# Sales Skills

A reusable, industry-neutral collection of Codex Skills for sales onboarding, prospect research, optional email outreach, contact-list updates, follow-up, and reporting.

## What's included

- `sales-onboarding`: Guides a teammate through project setup, including contact-list preferences, email service configuration, and required business information.
- `sales-workflow`: Runs a checkpointed sales workflow covering research, qualification, outreach, record updates, follow-up, and reporting.
- `example-sales`: A starter implementation that can be copied and adapted for a specific company.

## Installation

1. Download or clone this repository.
2. Copy the contents of the `skills` directory into your Codex Skills directory:

   ```text
   ~/.codex/skills/
   ```

3. The installed directory structure should look like this:

   ```text
   ~/.codex/skills/
   ├── sales-onboarding/
   ├── sales-workflow/
   └── implementations/example-sales/
   ```

See [DEPLOYMENT.md](DEPLOYMENT.md) for the complete setup instructions.

## Getting started

1. Run `sales-onboarding` and complete the guided setup.
2. Copy `implementations/example-sales` to create an implementation for your company.
3. Once the configuration status is `ready`, use the implementation to run `sales-workflow`.

Email delivery and contact-list writes require explicit configuration and authorization. The workflow never guesses contact email addresses or treats unconfirmed deliveries as successful.

## Configuration and security

- The YAML files in this repository contain example values only. They do not include real credentials or customer data.
- Never commit passwords, API keys, tokens, customer records, or runtime artifacts containing personal information.
- Commit only sanitized example configurations and keep real business configurations local.

## License

This project is licensed under the [MIT License](LICENSE).
