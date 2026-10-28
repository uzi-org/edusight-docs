# API reference

All partner endpoints live under `/api/v1/integrations` and require the `X-API-Key` header. Requests and responses are JSON. A live, machine-readable description is served by every EduSight deployment at `/api/v1/docs/openapi.json`.

## Conventions

- **IDs.** EduSight IDs are UUIDs. Your own learner IDs go in `external_student_id`. Wherever a learner is identified in a request body you may use either one.
- **Visibility.** A key sees only the learners its organisation registered, or the learners in the school the key is bound to. Requests for any other learner return `404 Not Found`, exactly as if the learner did not exist.
- **Errors.** `401` means a missing or invalid key. `404` means not found or not visible to you. `422` means validation failed, and the body says which field.

## Service

| Method and path | Purpose |
| --- | --- |
| `GET /health` | Confirms the key works. |
| `GET /model-version` | The model or rule set currently producing signals. Log it with your decisions. |

## Learners

### `POST /students`

Creates or updates one learner, matched on `external_student_id`.

| Field | Required | Notes |
| --- | --- | --- |
| `external_student_id` | yes | your ID |
| `full_name` | yes | |
| `grade_level` | yes | 1–13 |
| `age` | yes | 3–25 |
| `gender` | no | only where your policy and consent allow it |
| `external_school_id`, `school_name`, `country_code` | no | used when the key is not bound to a school |
| `enrollment_date` | no | defaults to today |

The response is the learner with EduSight's `id` and `created: true|false`.

### `POST /students/batch`

`{ "students": [ …up to 500 learner objects… ] }`. Each learner is applied independently. The response is `{ created, updated, failed, results: [{ external_student_id, ok, student | error }] }`.

## Information in

### `POST /events`

| Field | Required | Notes |
| --- | --- | --- |
| `event_type` | yes | `attendance`, `assessment`, `behavior`, `assignment`, `lms.activity`. Other types are acknowledged but not scored. |
| `event_id` | recommended | Makes the call idempotent per organisation. |
| `external_student_id` or `student_id` | yes, to persist | If the learner is unknown, a signal is returned but not stored. |
| `occurred_at` | no | ISO 8601 |
| `payload` | yes | any of the fields in [Mapping your data](index.md#mapping-your-data) |

Returns `202` with `{ accepted, duplicate, normalized_assessment, prediction, warnings }`.

### `POST /assessments`

The same fields as an event payload, sent at the top level, plus `assessment_date` and `notes`. Use it for term results or a full assessment. Returns `201` with `{ assessment_id, prediction }`.

### `POST /predict`

Stateless scoring: nothing is stored. Takes the same fields as an assessment and returns a support signal.

## Signals out

### `GET /students/{id}/risk`

The learner's latest support signal, or `prediction: null` with a message if there is none yet.

### `GET /students/{id}/recommendations`

`recommended_actions`, `support_needs`, `needs_human_review`, `teacher_summary`, `parent_summary` and `model_version`.

### The support signal object

```json
{
  "external_student_id": "sis-1042",
  "student_id": "6f1c…",
  "model_version": "rule-based-v1.0",
  "support_level": "urgent",
  "needs_human_review": true,
  "support_needs": [
    {
      "area": "learning_needs_screening",
      "title": "Consider a learning-needs screening",
      "reason": "The learner is attending but still struggling with core skills. Catch-up teaching may not be enough on its own.",
      "next_step": "Discuss with the special-needs coordinator or an educational assessor whether a screening (including vision and hearing) is appropriate. This is not a diagnosis.",
      "owner": "special_needs_coordinator",
      "review_in_days": 30,
      "referral_suggested": true
    }
  ],
  "suggested_intervention_plan": [
    { "step": 1, "action": "…", "owner": "teacher", "review_in_days": 14, "area": "academic" }
  ],
  "risk_drivers": [
    { "feature": "literacy_level", "label": "Foundational literacy support needed", "severity": "high", "value": 3, "recommendation": "…" }
  ],
  "recommended_actions": ["…"],
  "confidence": "medium",
  "data_completeness": 0.75,
  "missing_data_warnings": [{ "feature": "age", "recommendation": "Add age or date-of-birth-derived age." }],
  "teacher_explanation": "This is a decision-support signal for educators, not a final judgment about the learner. Review local context before acting.",
  "parent_explanation": "The school has identified practical support steps to help the learner stay engaged. …",
  "fairness_caution": "…",
  "risk_level": "high",
  "risk_probability": 0.68
}
```

`support_needs[].area` is one of `attendance`, `foundational_literacy`, `academic`, `wellbeing`, `learning_needs_screening` or `family_partnership`. `owner` is one of `teacher`, `counsellor` or `special_needs_coordinator`. New values may be added, so treat unknown values gracefully.

## Webhooks

| Method and path | Purpose |
| --- | --- |
| `POST /webhooks` | `{ url, event_types? }`. Returns the subscription and its `secret`, shown once. |
| `GET /webhooks` | Your subscriptions. |
| `DELETE /webhooks/{id}` | Deactivates a subscription. |
| `GET /webhooks/{id}/deliveries` | The last 100 deliveries, with status, attempts and last error. |
| `POST /api/v1/webhooks/test` | Queues a signed `webhook.test` event to your subscriptions that accept it. |

See [Webhooks](webhooks.md) for payloads and signature checking.
