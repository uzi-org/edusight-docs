# Responsible use

EduSight holds information about children. Our job is to help those children, and that sets firm limits on how EduSight is built and how it may be used.

## Our principles

1. **Support, never sanction.** EduSight exists to get help to learners sooner. It must not be used to punish, exclude, stream, rank or refuse admission to a learner.
2. **People decide.** EduSight suggests. Teachers, counsellors, school leaders and families decide. Urgent cases and referrals are always marked for human review.
3. **Explain everything.** Every signal comes with the reasons behind it and how confident EduSight is. A signal with low confidence or missing data says so.
4. **Language that respects the learner.** Wording meant for families and learners is supportive and never includes risk scores or labels. EduSight never describes a child as a "likely dropout" or anything similar.
5. **Not a diagnosis.** A suggestion to screen for learning needs, or to involve a counsellor, means a qualified person should take a closer look. It never means the learner has a condition.
6. **Collect only what helps.** EduSight works with the information schools already have. It does not need sensitive attributes to produce a signal.

## Sensitive information

EduSight does not use the following to produce a learner's signal: ethnicity, religion, health, disability, pregnancy, household income, orphan or vulnerability status, or refugee status. Where the law and consent allow, some of these may be used **only** in aggregate, to check that signals are fair across groups. They are never used to make a decision about an individual.

## Data protection

- **Isolation.** Each partner organisation sees only its own learners.
- **Minimal webhooks.** Notifications carry identifiers and support information, never names or scores.
- **Keys and secrets.** API keys are stored as one-way hashes and shown once. Webhook payloads are signed so you can verify they came from EduSight.
- **Encryption in transit.** EduSight requires https for webhook delivery. Deployments should serve the API over TLS.
- **Legal basis.** Every deployment must comply with the data protection law where it operates. Examples include Kenya's Data Protection Act 2019, South Africa's POPIA, the GDPR where it applies, and national education data rules. Requirements are reviewed for each country before launch.

## What we ask of partners

If you integrate EduSight into your product, you agree to:

- show `support_level` and `support_needs` in place of raw probabilities, and never show risk scores to learners or families;
- route every learner with `needs_human_review: true` to a named person, and never trigger automatic consequences from a signal;
- record which staff member viewed and acted on a recommendation, and store the `model_version` with it;
- send only the data you are permitted to share, and honour deletion requests from schools and families;
- tell us about any harm or misuse you become aware of.

## Known limitations

We publish these openly because they matter to how much weight a signal deserves.

- **Not yet validated on real school data.** EduSight's signals come from transparent, rule-based logic. Its trained models were trained on **synthetic** data. Neither has yet been evaluated against real learner outcomes in African schools. Treat every signal as a prompt to look closer, not as a prediction you can rely on alone.
- **Fairness has not yet been measured on real populations.** That work needs real, consented data from partner schools.
- **Thresholds are general.** The thresholds behind the support needs, such as attendance below 85% or a score below 50%, are general defaults. Schools and systems differ, and per-school configuration is planned.
