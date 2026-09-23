# Development Workflow

## Repository

Main repository:

```text
baithul-madeena/
```

Submodules:

```text
backend/
frontend/
```

Clone with submodules:

```bash
git clone --recurse-submodules <main-repository-url>
```

Initialize existing clone:

```bash
git submodule update --init --recursive
```

---

## Branching

Use the team's chosen Git strategy.

Recommended principle:

- one coherent feature/fix per branch;
- keep backend/frontend changes coordinated;
- update parent repository submodule SHAs only after submodule commits exist remotely.

---

## Cross-Submodule Changes

When a feature changes API and UI:

1. implement/update backend contract;
2. test backend;
3. implement frontend against contract;
4. test frontend;
5. update main repository documentation if needed;
6. commit submodules;
7. update main repository submodule pointers.

---

## API Contract Changes

Before making a breaking API change:

- identify frontend consumers;
- document migration;
- prefer additive changes;
- version if necessary.

---

## Database Migration Review

Check:

- reversibility;
- default values;
- nullability;
- data backfill;
- index impact;
- branch integrity;
- production locking risk.

---

## Definition of Done

A change is complete when applicable:

```text
[ ] backend implementation complete
[ ] backend tests pass
[ ] frontend implementation complete
[ ] frontend tests pass
[ ] branch isolation verified
[ ] authorization verified
[ ] financial/stock invariants verified
[ ] migration reviewed
[ ] docs updated
[ ] no secrets added
[ ] API compatibility considered
```

---

## Commit Guidance

Prefer commits that explain business intent.

Examples:

```text
feat(agreements): add tenant installment schedule
fix(branch): prevent cross-branch property lookup
feat(inventory): record scrap as stock movement
fix(payments): prevent duplicate cash receipt posting
```

---

## Review Checklist

Reviewers should ask:

1. Can this leak another branch's data?
2. Can a user submit another branch's foreign key?
3. Can this duplicate a financial posting?
4. Can this make stock inconsistent?
5. Can this double-book a Property?
6. Is the state transition legal?
7. Are calculations server-authoritative?
8. Are tests present for failure paths?
## Phase 1 release verification

Before a release candidate is promoted, run the backend feature suite, Pint,
Composer audit, frontend test suite, production build, and NPM audit. Then run
the [Phase 1 release smoke checklist](release-checklist.md) against the deployed
configuration. Do not treat frontend visibility as authorization; verify branch
scope and permissions through the API as well.
