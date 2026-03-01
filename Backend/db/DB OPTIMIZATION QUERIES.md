

**One-liner:** Don't call LOWER() inside SQL/JPQL — do it in the service layer before the query runs.

## Why It Matters
`LOWER(column) = :value` in a WHERE clause makes the query **non-sargable** —
the database cannot use an index on that column and will do a full table scan.
At scale, this kills performance silently.

## Core Concept
```
// ❌ Non-sargable — index on `name` is ignored
WHERE LOWER(e.name) = :name

// ✅ Sargable — index is used
WHERE e.name = :name  // with name already lowercased in Java
```

Normalize the input in the service, not the database:
```java
// Service layer
public Post findBySlug(String slug) {
    return postRepository.findBySlug(slug.toLowerCase().trim());
}
```

## Gotchas
- This only works if your stored data is already normalized (all lowercase).
  Make sure you lowercase on **write** too, not just on **read**
- If stored data is mixed case, you need a **functional index** in Postgres:
  `CREATE INDEX idx_lower_name ON posts (LOWER(name));`
- JPQL and native queries both have this problem — neither is immune

## Tags
#backend #database #performance #spring-boot #jpql

## Links
- [[Functional Indexes in PostgreSQL]]
- [[Service Layer Responsibilities]]
- [[The LEFT JOIN + WHERE Trap (Silent INNER JOIN)]]