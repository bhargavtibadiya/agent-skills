---
name: coding-guidelines
description: Use this skill whenever writing, reviewing, or refactoring backend TypeScript/Node.js code. Enforces 17 mandatory rules covering code structure, service/controller separation, database integrity, Swagger sync, Express routing, strict TypeScript typing, error handling, form-data coercion, and import style. Activate whenever the user touches controllers, services, routes, DTOs, or any backend logic.
license: MIT
metadata:
  author: bhargavtibadiya
  version: "1.0.1"
---

# Coding Guidelines

Mandatory rules for all backend TypeScript/Node.js code. Every rule applies by default — no exceptions unless explicitly noted.

---

## Quick Reference

### Code Structure

| # | Rule |
|---|------|
| 1 | Controllers orchestrate only — all business logic lives in services |
| 2 | Never use IIFEs — use named async functions |
| 3 | No magic strings or numbers — define all literals and constants in a constants file |

### Database Patterns

| # | Rule | Ref |
|---|------|-----|
| 4 | All DB queries must include `{ isDeleted: false }` unless querying deleted records explicitly | [details](references/database-patterns.md#4-soft-delete-handling) |
| 5 | Always validate ID existence before any create/update/delete — return 404 if not found | [details](references/database-patterns.md#5-id-existence-validation) |
| 6 | Multi-step DB writes must be wrapped in a transaction | [details](references/database-patterns.md#6-database-transactions) |

### API & Routing

| # | Rule | Ref |
|---|------|-----|
| 7 | Updating any API requires syncing Swagger at `src/services/swagger/` | [details](references/api-and-routing.md#7-swagger-sync) |
| 8 | In Express routers, specific routes must come before dynamic `/:param` routes | [details](references/api-and-routing.md#8-express-route-ordering) |
| 9 | Use centralised `ROUTES` constants — never hardcode path strings in routers | [details](references/api-and-routing.md#9-route-structure) |
| 10 | All admin APIs must be grouped with `Admin - <Section>` tags in Swagger | [details](references/api-and-routing.md#10-admin-api-organization) |

### TypeScript Patterns

| # | Rule | Ref |
|---|------|-----|
| 11 | Never use `as unknown`, `as any`, or bare `as SomeType` — use type narrowing | [details](references/typescript-patterns.md#11-no-type-casting) |
| 12 | Type request body via `const dto: DtoType = req.body` after `validateDto` — no `as` casts | [details](references/typescript-patterns.md#12-request-body-and-params-typing) |
| 13 | All number fields in form-data DTOs must use `z.coerce.number()` | [details](references/typescript-patterns.md#13-form-data-number-coercion) |
| 14 | Always use single-line imports | [details](references/typescript-patterns.md#14-import-statement-style) |
| 15 | Build response objects in a named variable first when more than 2 keys | [details](references/typescript-patterns.md#15-controller-response-data) |
| 16 | Prepare error data in a variable first, then pass to the response helper | [details](references/typescript-patterns.md#16-error-handling) |
| 17 | Never swallow errors silently — always rethrow, log+rethrow, or convert to a domain error | [details](references/typescript-patterns.md#17-async-error-propagation) |

---

## Reference Files

- **Database Patterns** (rules 4–6) → [references/database-patterns.md](references/database-patterns.md)
- **API & Routing** (rules 7–10) → [references/api-and-routing.md](references/api-and-routing.md)
- **TypeScript Patterns** (rules 11–17) → [references/typescript-patterns.md](references/typescript-patterns.md)

---

## When Reviewing Code

Check every rule that applies to the change. Flag violations with the rule number (e.g. "Rule 4 violation: missing `isDeleted: false` filter"). Suggest the correct pattern from the reference file.
