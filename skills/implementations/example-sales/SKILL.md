---
name: example-sales
description: Configure and run the reusable sales workflow for a new company. Use as the starter implementation when a teammate needs prospecting, optional Contact List updates, optional Agent Mail outreach, and reporting without inheriting another company's configuration.
---

# Example Sales Implementation

Start with `config.yaml`. Run `../../sales-onboarding/SKILL.md` until `setup.status` is `ready`, then run `../../sales-workflow/SKILL.md`.

## Rule precedence

Apply explicit user instruction and this implementation configuration before the generic skills. If an external reference conflicts with the user's or this implementation's rules, the user's and implementation rules prevail.

## Bindings

```yaml
workflow: ../../sales-workflow/SKILL.md
onboarding_workflow: ../../sales-onboarding/SKILL.md
configuration: ./config.yaml
configuration_example: ../../sales-workflow/config.example.yaml
```

## Setup behavior

- Retain every prospect with a usable contact email; do not use quality or confidence as a rejection gate.
- Use Agent Mail only after it is connected and verified. If the user explicitly disables it after final confirmation, run prospecting without outreach.
- Update a Contact List only after the user confirms an existing filename or approves a new update folder. If they explicitly disable it after final confirmation, never auto-fill a file.
- Keep company facts, credentials, sender identity, product information, local file paths, and tracker/report paths only in `config.yaml`.

