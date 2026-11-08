# Tenant owner-coverage override

## Intent

Allow an authorized user to create a tenant agreement whose dates exceed the
selected owner agreement, but require an explicit override flag and a reason.
The normal coverage rule remains unchanged for everyone else.

## Scope

- Add a dedicated `agreements.override_coverage` permission.
- Add `coverage_override` and `coverage_override_reason` to tenant-agreement
  creation only.
- Require the permission and a non-empty reason when the override is requested.
- Return a non-blocking warning in the successful create response indicating
  that owner coverage was overridden.
- Record the actor, branch, reason, source owner agreement, tenant dates, and
  owner dates in an append-only audit event.
- Add backend feature tests for rejection without the override, rejection
  without permission/reason, successful authorized override, and audit data.
- Update the Angular creation form with a warning/confirmation dialog and
  reason field. The frontend remains a UX layer; backend authorization is
  authoritative.
- Update the API schema documentation and permission seed/migration.

## Non-goals

- Do not change property occupancy overlap protection.
- Do not allow override through tenant PATCH, approval, extension, or renewal
  unless separately requested.
- Do not automatically extend the owner agreement.
- Do not silently alter payment schedules.

## Approach

Use the existing `PropertyAvailabilityService::assertOwnerCoverage` call with
an explicit `allowOutsideCoverage` argument from the tenant create workflow.
When the flag is true, the service still verifies the source agreement,
property relationship, branch, and usable status, but skips only the date
boundary failure. The controller checks `agreements.override_coverage` and the
reason before entering the transaction, then records a dedicated audit event
after the tenant agreement is saved.

The create response will include a top-level `warnings` array containing a
stable warning code and message when the override was used. Existing normal
responses remain unchanged.

## Open implementation call

The existing role seeder and management-permission migration will grant the new
permission to the current system roles, matching the repository's existing
permission bootstrap behavior. Administrators can then remove it from roles
that should not bypass owner coverage.
