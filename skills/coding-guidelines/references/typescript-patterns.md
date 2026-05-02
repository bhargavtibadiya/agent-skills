# TypeScript Patterns

---

## 11. No Type Casting

**Never use `as unknown`, `as any`, or a bare `as SomeType` cast** anywhere in the codebase. These bypass TypeScript's type system and hide real bugs.

Use proper type narrowing: `typeof`, `instanceof`, Zod validation, or explicit variable assignment after validation middleware.

```typescript
// ❌ Forbidden — all of these
const query = req.query as unknown as ListQueryType;
const dto = req.body as SignupDtoType;
const id = req.params.id as string;
const user = result as UserType;

// ✅ Body — explicit variable after validateDto middleware
const dto: SignupDtoType = req.body;

// ✅ Params — narrow with typeof
const id = req.params.id;
if (typeof id !== 'string' || !id) {
  sendBadRequest(res, 'Invalid ID parameter');
  return;
}

// ✅ Query — parse and validate each field individually
const page = typeof req.query.page === 'string' ? parseInt(req.query.page, 10) : 1;
const search = typeof req.query.search === 'string' ? req.query.search.trim() : undefined;

// ✅ Unknown value — use Zod to parse and get a typed result
const parsed = MySchema.safeParse(value);
if (!parsed.success) { /* handle */ }
const typed = parsed.data; // fully typed, no cast
```

**`req.query` values are always `string | string[] | ParsedQs | undefined`** — never cast them; parse each field explicitly.

---

## 12. Request Body and Params Typing

**Request body:** after `validateDto(SomeDto)` middleware runs, `req.body` is already validated. Assign it with an explicit type annotation — do not cast.

```typescript
// ✅ Correct
const dto: SignupDtoType = req.body;

// ❌ Avoid
const dto = req.body as SignupDtoType;
const dto: any = req.body;
```

**Route params:** Express types `req.params.x` as `string | string[]`. Always narrow before use — never assume it's a string.

```typescript
// ✅ Correct
const { sessionId } = req.params;
if (typeof sessionId !== 'string' || !sessionId) {
  sendBadRequest(res, 'Invalid session ID');
  return;
}
// sessionId is now narrowed to string

// ❌ Avoid
const sessionId = req.params.sessionId as string;
```

---

## 13. Form-Data Number Coercion

When a route uses multer middleware (`upload.single()`, `upload.array()`), **all form fields arrive as strings** regardless of their intended type. All number fields in those DTOs **must use `z.coerce.number()`** instead of `z.number()`.

```typescript
// ✅ Correct — DTO for a multer route
export const CreateDogDto = z.object({
  name: z.string().min(1),
  weight: z.coerce.number().positive().optional(),
  height: z.coerce.number().positive().optional(),
  lastSeenLatitude: z.coerce.number().min(-90).max(90).optional().nullable(),
  lastSeenLongitude: z.coerce.number().min(-180).max(180).optional().nullable(),
});

// ❌ Wrong — will fail at runtime: "expected number, received string"
export const CreateDogDto = z.object({
  weight: z.number().positive().optional(),
});
```

**When to apply:**
- ✅ DTOs validated on routes that use `upload.single()`, `upload.array()`, or any multer middleware
- ✅ Any endpoint accepting `multipart/form-data`
- ❌ DTOs used only with JSON (`application/json`) endpoints
- ❌ Query param DTOs (handled separately)

**How to identify:** if the route stack includes multer middleware AND calls `validateDto(SomeDto)`, all number fields in that DTO need `z.coerce.number()`.

**Common fields that need coercion:** `latitude`, `longitude`, `weight`, `height`, `maxParticipants`, `minParticipants`, any numeric measurement or count in a form-data payload.

---

## 14. Import Statement Style

**Always use single-line imports.** Split across lines only when the line exceeds 120 characters.

```typescript
// ✅ Correct
import { SafeUser, PaginatedUsersResponse, UserQueryFilters } from '../users/types/user.types.js';
import { sendSuccess, sendBadRequest, sendNotFound } from '../utils/response.js';

// ❌ Avoid — unnecessary multiline for short import lists
import type {
  SafeUser,
  PaginatedUsersResponse,
} from '../users/types/user.types.js';
```

Group imports in this order (with a blank line between groups):
1. Node built-ins (`node:path`, `node:fs`)
2. Third-party packages (`express`, `zod`, `prisma`)
3. Internal absolute paths (`~/modules/...`)
4. Relative imports (`../`, `./`)

---

## 15. Controller Response Data

**Do not pass inline objects** into response helpers (`sendSuccess`, `sendCreated`, `sendBadRequest`, etc.) when the object has more than two keys. Declare a variable first, then pass it. This keeps the response call readable and the data easy to trace.

```typescript
// ✅ Correct — named variable for non-trivial payload
const responseData = { user: result.user, token: result.token, expiresIn: result.expiresIn };
sendSuccess(res, 'Login successful', responseData);

// ✅ Fine inline — only one key
sendSuccess(res, 'Profile fetched', { profile });

// ❌ Avoid — multi-key inline object buried in a function call
sendSuccess(res, 'Login successful', { user: result.user, token: result.token, expiresIn: result.expiresIn });
```

---

## 16. Error Handling

Always handle errors in a **clean, flat, readable way**. Never write complex logic, mappings, or conditionals as inline arguments to response helpers.

**Pattern: prepare error data in a variable first, then pass it.**

```typescript
// ✅ Required pattern
const errors = dtoResult.error.issues.map((issue) => ({
  field: issue.path.join('.'),
  message: issue.message,
}));
sendBadRequest(res, 'Validation failed', errors);
return;

// ❌ Avoid — complex inline expression inside the response call
sendBadRequest(res, 'Validation failed', dtoResult.error.issues.map((i) => ({ field: i.path.join('.'), message: i.message })));
```

**This pattern is mandatory for all error types:** validation, database, params, business logic, and external API errors.

**Always `return` after sending an error response** — never let execution fall through to a success response below.

---

## 17. Async Error Propagation

**Never swallow errors silently.** Empty `catch` blocks and unhandled promise rejections hide bugs and make debugging impossible in production.

```typescript
// ❌ Forbidden — silently swallows the error
try {
  await doSomething();
} catch (_e) {}

// ❌ Forbidden — logs but swallows, caller has no idea it failed
try {
  await doSomething();
} catch (error) {
  console.error(error);
  // execution continues as if nothing happened
}

// ✅ Correct — rethrow so the caller can handle it
try {
  await doSomething();
} catch (error) {
  console.error('[ServiceName] doSomething failed:', error);
  throw error;
}

// ✅ Correct — convert to a typed domain error and rethrow
try {
  await externalApiCall();
} catch (error) {
  throw new AppError('EXTERNAL_API_FAILURE', 'Payment service unavailable', { cause: error });
}
```

**Rules:**
- Every `catch` block must either rethrow the error or throw a new domain error wrapping it.
- Log before rethrowing when the context is useful for debugging.
- Never use `void asyncFn()` unless you genuinely do not care about failure — and even then, attach a `.catch()`.
- Never use `Promise.all` without handling individual rejections when partial failure is meaningful.
