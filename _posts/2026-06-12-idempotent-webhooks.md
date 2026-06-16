---
title: "Designing an Idempotent Webhook Pipeline"
date: 2026-06-12
author: Manoj Kumar
category: "Architecture"
excerpt_override: "Webhooks retry. Your handlers must assume every event arrives more than once — here's the pattern we standardized on."
# image: /assets/img/webhooks.png
---

Every webhook provider retries on failure, which means your handler *will* see the
same event twice. If processing isn't idempotent, you get double charges, duplicate
emails, and corrupted state. Here's the pipeline we settled on.

## The core idea

Treat the provider's event ID as a unique key and record it the moment you start
processing. If you've seen it before, acknowledge and stop.

```sql
INSERT INTO processed_events (event_id, received_at)
VALUES ($1, now())
ON CONFLICT (event_id) DO NOTHING
RETURNING event_id;
```

If the `RETURNING` row is empty, it's a duplicate — return `200` immediately so the
provider stops retrying.

## The three stages

1. **Verify** the signature before trusting anything in the payload.
2. **Deduplicate** using the insert above, inside the same transaction as your work.
3. **Process** the side effect, then commit atomically.

The crucial detail is step 2 and 3 sharing **one transaction**. If processing fails,
the dedup row rolls back too, so a legitimate retry can succeed.

## Why not just check-then-insert?

Because two concurrent deliveries can both pass the check before either inserts. The
`ON CONFLICT` approach pushes uniqueness into the database, where it's actually safe
under concurrency.

This pattern has handled millions of events for us without a single duplicate-side-effect
incident since we shipped it.
