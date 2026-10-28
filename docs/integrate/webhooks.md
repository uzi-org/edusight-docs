# Webhooks

Webhooks let your system hear about changes as they happen, so you do not need to poll for every learner.

## Events

| Event | Sent when |
| --- | --- |
| `support_signal.changed` | A learner's support level changes. A learner's first signal is sent only if it is not `routine`. |
| `support_signal.referral_suggested` | A new referral appears for a learner, for example a learning-needs screening. |
| `webhook.test` | You call `POST /api/v1/webhooks/test`. |

Subscribe to all three, or choose with `event_types` when you create the subscription.

## Payload

```json
{
  "id": "4c1e9a5e-…",
  "type": "support_signal.changed",
  "created_at": "2026-10-28T09:14:03.120Z",
  "data": {
    "student_id": "6f1c…",
    "external_student_id": "sis-1042",
    "assessment_id": "b20d…",
    "support_level": "urgent",
    "previous_support_level": "watch",
    "needs_human_review": true,
    "support_needs": [
      { "area": "attendance", "owner": "teacher", "referral_suggested": false },
      { "area": "wellbeing", "owner": "counsellor", "referral_suggested": false }
    ],
    "model_version": "rule-based-v1.0"
  }
}
```

Webhooks carry **identifiers and support information only**. They never contain names, scores or probabilities. Fetch details over the authenticated API when your user opens the learner. `support_signal.referral_suggested` also includes `new_referral_areas`.

## Verifying the signature

Every request carries:

```http
X-EduSight-Signature: t=1793193442,v1=1545a3a86ce60feb…
X-EduSight-Event: support_signal.changed
X-EduSight-Delivery: 4c1e9a5e-…
```

`v1` is the hex HMAC-SHA256 of `"{t}." + raw request body`, keyed with your subscription secret. Reject the request if the signature does not match, or if `t` is more than five minutes old. Always verify against the **raw** body bytes, before any JSON parsing.

```python
from edusight_africa import verify_webhook_signature

ok = verify_webhook_signature(secret, request.headers["X-EduSight-Signature"], request.body)
```

```ts
import { verifyWebhookSignature } from '@edusight-africa/sdk';

const ok = await verifyWebhookSignature(secret, req.headers['x-edusight-signature'], rawBody);
```

## Delivery and retries

- Respond with any `2xx` within 10 seconds to acknowledge. Do the real work after acknowledging.
- Failures are retried with exponential backoff: 30 seconds, then 1, 2, 4… minutes, up to 6 hours apart. After 8 attempts the delivery is marked `failed`.
- Deliveries can arrive more than once and out of order. Use `id` to deduplicate, and `created_at` to order.
- Check `GET /api/v1/integrations/webhooks/{id}/deliveries` to see what was sent and why anything failed.

## Requirements for your endpoint

- It must use **https**.
- It must be reachable on the public internet. EduSight will not deliver to private, loopback or link-local addresses.
- Redirects are not followed.
