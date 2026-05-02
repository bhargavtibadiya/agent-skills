# API & Routing

---

## 7. Swagger Sync

Every time any part of an API changes — controller logic, route path, request body, query params, response shape, or config — **Swagger must be updated in the same PR/commit**.

- Swagger files live at: `src/services/swagger/`
- Keep request body schema, params, query params, response examples, and error responses in sync.
- If you encounter an existing Swagger entry that is incorrect or outdated, **fix it immediately** even if it is unrelated to your current task.

**What must stay in sync:**
- Endpoint path and HTTP method
- Request body shape (required fields, optional fields, types)
- Path and query parameters
- All documented response codes and their example payloads
- Auth requirements (`security` field)

---

## 8. Express Route Ordering

Express matches routes in the order they are defined. A dynamic segment like `/:id` will match **any string**, including paths intended for named routes declared later.

**Rule: always define specific/named routes before dynamic routes.**

```typescript
// ✅ Correct — specific routes first
router.get('/groups', categoryController.getGroups);
router.get('/subcategories', categoryController.getSubcategories);
router.get('/featured', categoryController.getFeatured);
router.get('/:categoryId', categoryController.getById);       // dynamic last

// ❌ Wrong — dynamic route shadows all specific routes below it
router.get('/:categoryId', categoryController.getById);
router.get('/groups', categoryController.getGroups);          // never reached
router.get('/subcategories', categoryController.getSubcategories); // never reached
```

**Common symptom:** a named route handler is never called, and the dynamic handler receives a string like `"groups"` as an ID. Fix by moving all specific routes above any `/:param` route.

---

## 9. Route Structure

- **Always define route paths in centralised `ROUTES` constants** — never hardcode strings directly in router files.
- **Action-specific routes** (e.g. `/leave`, `/approve`, `/bulk`) must be explicit named paths, not inlined `/:id/action` patterns.
- Apply the same ordering rule from Rule 8: action routes before dynamic routes.

```typescript
// src/utils/constants/routes.ts
export const ROUTES = {
  MEET_UPS: {
    BASE: '/meet-ups',
    LEAVE: '/:meetUpId/leave',
    APPROVE: '/:meetUpId/approve',
    BY_ID: '/:meetUpId',
  },
} as const;

// ✅ Correct — using constants
router.post(ROUTES.MEET_UPS.LEAVE, authenticate, meetUpController.leaveMeetUp);
router.patch(ROUTES.MEET_UPS.APPROVE, authenticate, meetUpController.approveMeetUp);
router.get(ROUTES.MEET_UPS.BY_ID, authenticate, meetUpController.getById);

// ❌ Wrong — hardcoded strings scattered in router files
router.post('/:meetUpId/leave', authenticate, meetUpController.leaveMeetUp);
router.put('/:userId/bulk', controller.bulkUpdate);
```

**Benefits:** find all routes with a single grep, rename a path in one place, catch typos at definition time.

---

## 10. Admin API Organization

All admin-facing endpoints must be **visually grouped in Swagger** using a consistent tag format: `Admin - <Section>`.

```yaml
# ✅ Correct
tags:
  - Admin - Listings

tags:
  - Admin - Users

tags:
  - Admin - Reports

# ❌ Wrong — mixed with public API tags or missing admin prefix
tags:
  - Listings

tags:
  - admin
```

**Rules:**
- Every admin endpoint needs the `Admin - <Section>` tag — no exceptions.
- Use title case for section names: `Admin - Meet Ups`, not `admin - meet ups`.
- When adding a new admin section, check whether a tag for it already exists and reuse it.
- Admin and public endpoints for the same resource must use **separate tags** so they appear in separate Swagger UI groups.
