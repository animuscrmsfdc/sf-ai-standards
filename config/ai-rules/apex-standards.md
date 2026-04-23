# Apex Coding Rules

Machine-readable reference for AI enforcement. Rules here are authoritative for code review, static analysis, and generation.

---

## Sharing Model

- **RULE**: Every Apex class must declare `with sharing` unless a documented exception exists.
- **EXCEPTION**: Classes that must run without sharing (e.g. certain system operations) must declare `without sharing` explicitly — never omit the keyword.
- **EXCEPTION**: Inner classes inherit the sharing from the outer class — no separate declaration needed.

## FLS Enforcement

- **RULE**: All @AuraEnabled methods that write records must call `Security.stripInaccessible()` before DML.
- **RULE**: Use `AccessType.CREATABLE` for inserts, `AccessType.UPDATABLE` for updates. Never use `AccessType.UPSERTABLE` on objects with master-detail fields.
- **RULE**: `WITH SECURITY_ENFORCED` is permitted only on single-object SOQL queries with no relationship traversal (no `Parent__r.Field` patterns).
- **RULE**: `Security.stripInaccessible(AccessType.READABLE, results)` is required on query results returned from @AuraEnabled read methods when the data will be rendered in LWC.

## SOQL Safety

- **RULE**: Never concatenate user input into `Database.query()` — always use bind variables (`:variable`).
- **RULE**: Never place SOQL inside a `for` loop — collect IDs first, then query outside the loop.
- **RULE**: Prefer `WITH SECURITY_ENFORCED` or `stripInaccessible` over manual FLS checks in queries.

## DML Safety

- **RULE**: Never place DML statements inside a `for` loop.
- **RULE**: Always bulkify: operate on lists/collections, not single records.
- **RULE**: Check `isCreateable()`, `isUpdateable()`, or `isDeletable()` before DML in non-@AuraEnabled contexts where `stripInaccessible` is not used.

## Trigger Pattern

- **RULE**: One trigger per object. Trigger body contains only a `switch on Trigger.operationType` statement delegating to a handler class.
- **RULE**: Handler class uses `public static` methods per event (e.g. `beforeInsert`, `afterUpdate`).
- **RULE**: Recursion control via a `static Boolean hasRun = false` flag on the handler class.
- **PROHIBITED**: Do not use the Kevin O'Hara TriggerHandler base class framework.

## Hardcoded Values

- **RULE**: Never hardcode record IDs or org-specific IDs in Apex or metadata.
- **RULE**: Use Custom Metadata Types for environment-agnostic configuration.
- **RULE**: Use Custom Labels for user-facing text and environment-specific values.

## Exception Handling

- **RULE**: Wrap all external calls (DML, `EventBus.publish`, callouts) in `try/catch(Exception e)`.
- **RULE**: Never swallow exceptions silently — log via `Exception_Log__e` platform event.
- **RULE**: Never expose raw exception messages to end users — use `AuraHandledException` in @AuraEnabled methods.
- **RULE**: Never rethrow inside a trigger on the same object used for error logging (risk of rollback loop).

## Platform Events

- **RULE**: Call `EventBus.publish()` once per transaction with a list — never inside a loop.
- **RULE**: Check `SaveResult` after publish and log failures. Collect all failures before logging (single publish call for failures).
- **RULE**: Cap LongTextArea content at 20,000 chars before serializing into a platform event payload field. Append `...[TRUNCATED]` when truncation occurs.

## Testing

- **RULE**: Minimum 90% code coverage for all Apex classes and triggers.
- **RULE**: Use `@TestSetup` for shared test data. Never create data in individual test methods.
- **RULE**: Never use `@IsTest(SeeAllData=true)`.
- **RULE**: Use `System.runAs()` for any test that asserts sharing or permission behavior.
- **REQUIRED SCENARIOS**: happy path, bulk (200 records), negative/invalid input, permission boundary.

## Cryptography

- **RULE**: SHA-256 minimum for any hash or token operation. MD5 is prohibited for security-sensitive use.
