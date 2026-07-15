# Sales Skills Deployment Guide

1. Download or clone this repository.
2. Copy the contents of the `skills` directory into your Codex Skills directory:

   ```text
   ~/.codex/skills/
   ```

   The resulting structure should include:

   ```text
   ~/.codex/skills/
   ├── sales-onboarding/
   ├── sales-workflow/
   └── implementations/example-sales/
   ```

3. Run `sales-onboarding` the first time you use the package. It will guide you through selecting a contact list, connecting or disabling Agent Mail, and providing the business information required by your enabled features.
4. After setup is complete, use the `example-sales` implementation as a starting point for your sales automation.

> [!IMPORTANT]
> Never commit passwords, secrets, credentials, or real customer data to a public repository.
