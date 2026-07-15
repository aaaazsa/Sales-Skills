---
name: sales-onboarding
description: Configure a sales implementation for a non-technical teammate by locating or setting up a contact list, connecting or explicitly disabling Agent Mail, collecting only feature-required business information in chat, and writing validated configuration. Use before a teammate runs a sales workflow or whenever setup state is incomplete.
---

# Sales Onboarding

## Rule precedence

Apply explicit user instruction and project implementation configuration before this workflow. If any external rule conflicts with your project rules, your project rules win.

Run before the first sales run and whenever `setup.status` is not `ready`. Use plain language; never require YAML, file paths, adapters, or schema knowledge.

**Input:** project folder, existing configuration, and requested features: `prospecting`, `contact_list_update`, `email_outreach`, `reporting`.

**Output:** `onboarding_receipt` with setup status, Contact List decision, email decision, collected fields, missing fields, and user confirmations. Do not start prospecting, email, or file writes before status is `ready`.

## Phase 1 — Resolve Contact List

**Goal:** Determine whether and where Contact List may be updated.

**Execution:** Search the project folder recursively for `.xlsx`, `.xls`, `.csv`, or `.tsv` filenames matching configured keywords (`contact`, `customer`, `client`, `lead`, `客户`, `联系人`). Show matches by filename only.

**Decision gate:**

1. If matches exist, ask the user to copy the exact filename into chat. Never select silently.
2. If no match exists, ask whether they already have a Contact List. If yes, invite them to place it in the project folder and copy its filename into chat.
3. If no file exists, ask whether to create a Contact List update folder.
4. If automatic filling is declined, ask once more: “请确认：您不需要系统自动填写 Contact List，对吗？” Only a second explicit decline disables it.

**Success:** Existing filename confirmed, new folder approved, or final disable confirmation recorded.

**Failure:** Required user choice missing, named file absent, or selected file unreadable. Block onboarding and state the next action.

**Metrics:** `contact_list_resolution_rate`, `file_discovery_count`, `user_confirmation_count`, `setup_duration_ms`.

## Phase 2 — Resolve Agent Mail

**Goal:** Default to a connected Agent Mail mailbox or obtain an explicit decision to connect or disable email.

**Decision gate:**

1. If Agent Mail is connected, use it as the default provider.
2. If not connected, ask whether the user wants to connect Agent Mail.
3. If yes, invite registration/sign-in at `https://agent.qq.com`, then provide this prompt:

   `请阅读 https://agent.qq.com/doc/cli-setup.md 文档，按照步骤为我安装并配置 Agent Mail CLI。`

   Confirm connection from a successful status signal; never assume it succeeded.
4. If declined, ask once more: “请确认：您不需要邮件服务，只需要寻客服务，对吗？” Only a second explicit decline sets `email.enabled: false` and disables sending.

**Success:** Connected mailbox verified or email disabled after final confirmation.

**Failure:** Binding incomplete, status unknown, or final confirmation absent. Block onboarding.

**Metrics:** `mail_connection_rate`, `mail_disable_rate`, `binding_completion_rate`, `human_intervention_count`, `setup_duration_ms`.

## Phase 3 — Collect Required Information

**Goal:** Collect only fields required by enabled features and update configuration from user-supplied values.

Put required fields first and label each `（必填）`; all fields below the required sections are optional. If only prospecting is enabled and email is disabled, salesperson name and email are optional.

```text
请复制下面内容并填写。上方为必填项；没有标注“（必填）”的均为选填项。

【必填项】
公司产品信息（必填）：
目标客户类型（必填）：
目标国家/地区（必填）：

【启用邮件服务时必填】
公司名称（必填）：
业务员名字（必填）：
联系邮箱（必填）：

【启用 Contact List 自动更新时必填】
Contact List 文件名或新建文件夹名称（必填）：

【选填项】
公司官网：
业务员职位：
电话/WhatsApp：
产品卖点：
最低起订量：
邮件语气/语言：
竞品品牌：
其他备注：
```

**Success:** All enabled-feature requirements are present and configuration validates.

**Failure:** A required value is declined or validation fails. Offer to disable the dependent feature; otherwise block onboarding.

**Metrics:** `required_field_completion_rate`, `configuration_validation_success`, `feature_disable_rate`, `setup_duration_ms`.

## Handoff

With email enabled, start `sales-workflow`. With email disabled, skip outreach/send and record `Prospecting` or `Not Sent`, never `Sent`. With Contact List disabled, never create or modify it.

