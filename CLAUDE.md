# Salesforce Project — AI Coding Standards

## Setup (required for each project adopting this template)

Before using Claude Code with this project, replace the following placeholders in this file:

| Placeholder | Replace with |
|---|---|
| `<YOUR_ORG_ALIAS>` | Your Salesforce org alias (e.g. `my-sandbox`) |
| `<YOUR_SANDBOX_USERNAME>` | Your sandbox login username (e.g. `user@domain.com.sandbox`) |
| `<YOUR_PROJECT_NAME>` | The project name (e.g. `MYPROJECT`) |

Set `SF_TARGET_ORG` locally by adding it to your `.claude/settings.local.json`:
```json
{ "env": { "SF_TARGET_ORG": "<YOUR_ORG_ALIAS>" } }
```

See `CLAUDE.local.md` for additional local overrides you can make without affecting the shared config.

---

## Project

- Org: Salesforce Sandbox
- Project name: `<YOUR_PROJECT_NAME>`
- Default package directory: `force-app/main/default`
- API Version: 66.0
- Login URL: https://test.salesforce.com

## CLI

- Use `sf` CLI (not `sfdx`) for all Salesforce commands
- Default target org alias: `<YOUR_ORG_ALIAS>`
- Username: `<YOUR_SANDBOX_USERNAME>`
- Deploy: `sf project deploy start --source-dir force-app --target-org <YOUR_ORG_ALIAS>`
- Retrieve: `sf project retrieve start --source-dir force-app --target-org <YOUR_ORG_ALIAS>`

## Metadata Conventions

- Custom object API names: PascalCase + `__c` suffix (e.g. `SalesOrder__c`)
- Custom field API names: PascalCase + `__c` suffix (e.g. `TotalAmount__c`)
- Platform event API names: PascalCase + `__e` suffix (e.g. `OrderPlaced__e`)
- Validation rule names: Descriptive_Snake_Case (e.g. `Amount_Cannot_Be_Negative`)
- Always include a description on every metadata component
- Label and API name must match (no abbreviations)

## General

- Never guess or fabricate Salesforce API names, field names, or metadata — read the files first.

## Security Defaults

- Always use `with sharing` in Apex unless explicitly told otherwise.
- Never hardcode record IDs or org IDs — use Custom Metadata or Custom Labels.
- Never use `Database.query()` with string concatenation — always use bind variables.

## Apex Coding Standards

- Never use `WITH SECURITY_ENFORCED` on queries that include cross-object relationship fields (e.g. `Speaker__r.Name`) or as post-upsert return queries — use `Security.stripInaccessible(AccessType.READABLE, results).getRecords()` instead; it strips inaccessible fields gracefully rather than throwing
- `WITH SECURITY_ENFORCED` is safe only for simple single-object queries with no relationship traversal (e.g. `SELECT Id, Name FROM Session__c`)
- Never use `AccessType.UPSERTABLE` with `stripInaccessible` on junction objects that have master-detail fields — master-detail fields are creatable but not updatable, so UPSERTABLE strips them and causes DML failures; use `ss.Id == null ? AccessType.CREATABLE : AccessType.UPDATABLE` instead
- Always call `Security.stripInaccessible()` in @AuraEnabled write methods (insert/update/upsert)
- Triggers: one trigger per object, delegate logic to a handler class
- Bulkify all trigger and batch logic

## Testing

- Minimum 90% code coverage for all Apex
- Use `@TestSetup` for shared test data
- Never use `SeeAllData=true`
- Run tests: `sf apex run test --target-org <YOUR_ORG_ALIAS> --code-coverage`

## Git Branch Conventions

- Feature branches: `feature/<spec-slug>` where `<spec-slug>` matches the spec filename (e.g. spec `specs/order-management.md` → branch `feature/order-management`)
- Hotfix branches: `hotfix/<short-description>`
- Used by `/deploy`, `/run-tests`, and `/changelog` to link branch to spec

## Local Overrides

Personal overrides (org credentials, sandbox aliases, local tool preferences) go in `CLAUDE.local.md`, which is gitignored. See that file for the template.
