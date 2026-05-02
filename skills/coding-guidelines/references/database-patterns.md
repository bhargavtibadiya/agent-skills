# Database Patterns

---

## 4. Soft Delete Handling

All soft-deletable models have an `isDeleted` boolean field. **Every query against such a model must filter it out** unless the intent is explicitly to retrieve deleted records (e.g. admin restore flows).

**Applies to all queries** — `findMany`, `findFirst`, `findUnique`, `count`, `aggregate`, and any raw queries.

Models with soft delete include: `User`, `UserProfile`, `BusinessProfile`, `Dog`, `Listing`, `Category`, `ListingClaim` (and any new model you add with `isDeleted`).

```typescript
// ✅ Correct
const profile = await prisma.businessProfile.findFirst({
  where: { ownerId, isDeleted: false },
});

const listings = await prisma.listing.findMany({
  where: { categoryId, isDeleted: false },
});

// ❌ Wrong — soft-deleted records leak into results
const profile = await prisma.businessProfile.findFirst({
  where: { ownerId },
});
```

**For delete operations:** always use soft delete (`update` with `{ isDeleted: true }`) unless a hard delete is explicitly required and approved. Never call `prisma.model.delete()` without confirmation that hard delete is intended.

---

## 5. ID Existence Validation

If an ID is accepted from params, body, or query, **always verify the record exists** in the database before proceeding with any create, update, delete, or relation logic.

```typescript
// ✅ Correct
const listing = await prisma.listing.findFirst({
  where: { id: listingId, isDeleted: false },
});
if (!listing) {
  sendNotFound(res, 'Listing not found');
  return;
}
// safe to proceed

// ❌ Wrong — proceeding without existence check
await prisma.listingClaim.create({
  data: { listingId, userId },
});
```

**Rules:**
- Use `findFirst` (not `findUnique`) so you can include the `isDeleted: false` filter at the same time.
- Return **404** with a human-readable message — never a 500 or a generic error.
- Validate every foreign key ID in a request, not just the primary subject.

---

## 6. Database Transactions

Any operation that writes to more than one table, or performs multiple dependent writes, **must be wrapped in a Prisma transaction**. If any step fails, all steps must roll back automatically.

```typescript
// ✅ Correct — multi-step write in a transaction
const result = await prisma.$transaction(async (tx) => {
  const claim = await tx.listingClaim.create({
    data: { listingId, userId, status: 'PENDING' },
  });

  await tx.listing.update({
    where: { id: listingId },
    data: { claimCount: { increment: 1 } },
  });

  return claim;
});

// ❌ Wrong — two separate awaits with no rollback safety
const claim = await prisma.listingClaim.create({ data: { listingId, userId } });
await prisma.listing.update({ where: { id: listingId }, data: { claimCount: { increment: 1 } } });
// if the second call throws, the claim record is orphaned
```

**When a transaction is required:**
- Creating a record AND updating a related record's counter or status
- Deleting a parent AND cleaning up children
- Any sequence of writes where partial success leaves data in an inconsistent state

**Use `tx` inside the callback, not the global `prisma` client**, so all operations share the same transaction context.
