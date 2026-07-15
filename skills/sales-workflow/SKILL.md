---
name: sales-workflow
description: "Execute a reusable, checkpointed sales and customer-development workflow: research leads, retain contacts with usable emails, generate and send outreach when enabled, update records, follow up, and report results. Use with a project implementation configuration after sales-onboarding is ready."
---

# Sales Workflow

## Rule precedence

Apply rules in this order: explicit user instruction → project implementation `SKILL.md` and configuration → this workflow → examples or external references. If any conflict exists, the project implementation and user rules win. Do not change business rules from an external reference.

## Runtime contract

Require `config`, `run_id`, `implementation_id`, and `run_store`. Persist an artifact and evaluation event after each phase. Use `lead_id` as the entity key and `run_id:phase_id:lead_id` as the idempotency key. Resume at the first incomplete or invalid phase; never repeat a confirmed external write.

```yaml
phase_result:
  phase_id: string
  status: completed|rejected|blocked|failed
  artifact_ref: string
  next_phase: string|null
  error_code: string|null
  evaluation_event: {duration_ms: integer, cost: object|null, human_intervention: boolean, metrics: object}
```

Run only after `setup.status: ready`. If email is disabled, skip Phases 3–4 and use `Prospecting` or `Not Sent`, never `Sent`. If Contact List is disabled, never create, modify, or fill it.

## Industry-neutral design

Keep this workflow independent of any particular product, market, channel, or CRM schema. Resolve industry-specific meaning only from the implementation's product, industry, qualification, communication, and record modules.

- Treat the offering as a configurable `product_or_service`, not necessarily a physical product.
- Treat the target as a configurable `account`, `contact`, `partner`, or `opportunity`.
- Store universal fields first: identity, market, channel, source, contact route, fit evidence, status, owner, next action, and provenance.
- Keep industry-specific fields in implementation configuration; never add hard-coded projector, hardware, SaaS, medical, financial, or other vertical assumptions here.
- Separate `fit`, `email_quality`, `send_status`, `engagement_status`, and `commercial_stage`. One must not silently substitute for another.

## Contact-quality and decision-maker research

Do not treat any public email as equally useful for sales outreach. Apply this priority order:

1. A named decision-maker or buyer with a verified role and professional email.
2. An official `sales@`, `business@`, `partnerships@`, `commercial@`, `采购`, or `招商` address on the company website.
3. An official company-level `info@`, `enquiry@`, or `contact@` address when no better route is available.
4. `support@`, `service@`, `help@`, `hello@`, and generic contact forms only as research clues, never as default sales recipients.

When the website exposes only a support/service route:

- Search the company's official team, about, leadership, partner, dealer, press, and contact pages.
- Search public professional sources such as LinkedIn, trade-association pages, conference speaker pages, company announcements, and reputable business directories for roles such as Founder, CEO, Managing Director, Commercial Director, Head of Sales, Purchasing Manager, Category Manager, Buyer, or Business Development.
- Record the person's name, role, company association, source URL, and confidence. Do not infer a role from a name or guess an email pattern.
- Use a named professional email only when it is publicly attributable and matches the company domain or a reliable professional source.
- If only a person's name and role are found but no reliable email exists, keep the lead as `decision_maker_found_no_email`; do not send. Add it to the configured `manual_contact_lookup` queue and request that the customer manually provide or locate the decision-maker email.
- If neither a credible sales route nor a credible decision-maker route can be verified, exclude the lead from the outreach list while optionally retaining it in a research queue.

Required lead fields for outreach should include `contact_name`, `contact_role`, `contact_email`, `email_type`, `contact_source_url`, `role_source_url`, and `contact_confidence` whenever available.

## Manual contact lookup queue

For every `decision_maker_found_no_email` lead, persist `manual_contact_lookup_request` with company name, website, decision-maker name, role, company association, contact/role source URLs, and the explicit request: “请协助提供或查找该负责人的可靠工作邮箱；系统不会猜测邮箱格式或自动发送。”

- Set its list state to `waiting_for_user`, not `rejected` or `sent`.
- At list completion, group all pending requests into one customer-facing lookup list through the configured `manual_contact_lookup.delivery` channel. Do not send the lookup list unless that channel is configured and authorized; otherwise present it in the current customer chat.
- When the customer supplies an email, validate it as provided input, attach its source as `customer_provided`, and resume at Phase 3 for that lead. Do not infer an email from a name/domain combination.
- If the customer cannot provide an email, retain the record as `decision_maker_found_no_email` and close that outreach attempt without deletion.

## List-level orchestration

When the user provides a list or asks to process a batch, maintain a list run with one final state per candidate: `sent`, `failed`, `skipped`, `waiting_for_user`, or `blocked`.

1. Process candidates one at a time with a stable `run_id:lead_id` idempotency key.
2. Persist each research, draft, send, and error artifact immediately so the run can resume safely.
3. Do not declare the list complete while any candidate lacks a final state.
4. After every candidate has a final state, run `List Completion`:
   - sync only verified successful sends to the final Contact List;
   - deduplicate by the implementation's configured identity key, normally company plus email;
   - add source/provenance records for every researched candidate;
   - deliver or present the grouped manual-contact lookup list for `waiting_for_user` leads;
   - reconcile list totals and reporting totals;
   - read back and validate all writes before completion.
5. Never mark drafted, unconfirmed, CAPTCHA-blocked, authorization-blocked, or ambiguous messages as sent.

The Contact List may receive a staging activity event earlier when required for resume/idempotency, but final customer/contact status must be written only by the configured list-completion policy.

## Knowledge, template, and tool rules

- Retain every candidate with a credible sales route or verified decision-maker route. Do not reject solely for incomplete optional descriptive fields or priority; do reject support-only/no-route leads from outreach according to the contact-quality policy below.
- Use only configured product features, use cases, customer fit, competitor context, and allowed claims. Never invent product facts or improve positioning.
- Use configured industry channels, markets, terminology, and research-source guidance without changing search strategy.
- Render the configured email subject/body and resolve required placeholders. Validate configured tone, sender identity, required sections, and claim support. Never fabricate names, relationships, facts, or consent.
- Use configured adapters for research, email, records, checkpoints, and reporting. Normalize provider errors to `unauthorized|unavailable|timeout|rate_limited|human_challenge|ambiguous|validation_failed|unknown`; preserve provenance, redact secrets, and verify external writes.

## Phase 1 — Research leads

**Input:** target market, research constraints, configured product/industry context, prior-lead index, research adapter.

**Output:** `lead_candidate` with contact email, identity fields when available, market, channel, website, evidence, and provenance.

**Success:** At least one contact email exists and the candidate is not a duplicate under the configured key.

**Failure:** No credible sales route or decision-maker route after configured recovery or research adapter retry is exhausted.

**Metrics:** `research_success`, `email_found_rate`, `required_field_coverage`, `duplicate_rate`, duration, cost, intervention.

**Next:** Credible sales route or decision-maker route → Phase 2; support-only route → Phase 1 decision-maker recovery; no credible route → reject from outreach.

## Phase 2 — Retain and record lead

**Input:** `lead_candidate`, `lead_retention` configuration.

**Output:** `lead_evaluation` with `contact_email`, `email_status`, recorded descriptive fields, `decision`, optional priority, reasons, and policy version.

**Success:** A credible sales route or decision-maker route is recorded, with role/source/confidence captured when available. A support-only route is not sufficient for outreach unless the implementation explicitly enables it for a specific business reason.

**Failure:** Email is absent/malformed, contact role cannot be supported, or retention configuration is invalid.

**Metrics:** `lead_retention`, `email_format_validity`, `descriptive_field_coverage`, `retention_rate`.

**Next:** Usable email → Phase 3 when email is enabled; otherwise Phase 5 for prospecting records. `decision_maker_found_no_email` → manual contact lookup queue with `waiting_for_user`. No credible route → reject from outreach.

## Phase 3 — Generate outreach

**Input:** retained lead, configured sender/company/product data, configured email template and communication rules.

**Output:** `outreach_draft` with recipient, subject, body, template version, variables used, claim-evidence map, and validation result.

**Success:** Required placeholders resolve; required sections, sender identity, and communication rules pass; all claims are supported.

**Failure:** Recipient or required variable missing, unsupported claim remains, or policy validation fails.

**Metrics:** `draft_success`, `template_compliance`, `supported_claim_ratio`, `variable_resolution_rate`.

**Next:** Valid draft → Phase 4; correctable issue → regenerate within retry limit; otherwise block.

## Phase 4 — Send communication

**Input:** validated draft, connected email adapter, authorization state, send policy, idempotency key.

**Output:** `send_receipt` with recipient, subject, timestamp, provider reference, verification signal, attempts, and status.

**Success:** Exactly one provider-confirmed send for the idempotency key and durable receipt.

**Failure:** Missing authorization, human challenge, exhausted retry, or unresolved ambiguous result.

**Metrics:** `send_success`, `delivery_verification_rate`, `attempt_count`, `duplicate_send_count`.

**Next:** Confirmed/definitively failed send → Phase 5. CAPTCHA or ambiguous result → block and verify before resend.

## Phase 5 — Record activity

**Input:** lead artifacts, send receipt or prospecting status, configured record/contact-list target, field mapping, idempotency key.

**Output:** `activity_record_receipt` with target, entity key, written fields, external reference, status, and read-back validation.

**Success:** Record matches source artifacts and actual status after read-back. Persist the activity receipt immediately; update the final Contact List according to `contact_list.sync_timing`, defaulting to `after_list_complete` for batch runs.

**Failure:** Target unavailable, schema validation fails, or write cannot be verified.

**Metrics:** `record_success`, `field_coverage`, `status_consistency`, `duplicate_record_count`.

**Next:** Verified record → Phase 6 or Phase 7; otherwise retry only the write. For a batch, continue until every candidate has a final state, then run List Completion.

## Phase 6 — Follow up

**Input:** verified record, follow-up policy, engagement state, email/record adapters.

**Output:** `follow_up_action` with due date, eligibility, action status, supporting receipt, and updated engagement state.

**Success:** Policy-compliant action is scheduled, sent, skipped, or escalated with reason.

**Failure:** Unknown engagement state, prohibited action, or unverified send/write.

**Metrics:** `follow_up_completion`, `follow_up_response_rate`, `policy_compliance`, `human_intervention_rate`.

**Next:** Completed/skipped → Phase 7; unresolved state → block.

## Phase 7 — Report results

**Input:** verified period artifacts, reporting policy, report adapter.

**Output:** `report_receipt` with period totals, workflow metrics, report reference, reconciliation, and read-back validation.

**Success:** Totals reconcile; sent/replies/samples/quotations/orders remain distinct; required metrics are complete.

**Failure:** Unresolved mismatch, missing metrics, or failed report read-back.

**Metrics:** `report_success`, `reconciliation_variance`, `metric_coverage`, `stage_accuracy`.

**Next:** Verified report → complete; mismatch → repair source data and rerun Phase 7 only.

## Completion

Return each lead's final status, list completion status, latest checkpoint, artifact references, recovery actions, reconciliation, and phase metrics. Mark unresolved optional data `Unknown`; never silently invent it.
