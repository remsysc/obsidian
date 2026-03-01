**One-liner:** Don't call LOWER() inside SQL/JPQL — normalize in the service layer before the query runs.

## Why It Matters
`LOWER(column) = :value` makes the query **non-sargable** — the database cannot use
an index on that column and will do a full table scan. At scale, this kills performance silently.

## Core Concept
```java
// ❌ Non-sargable — index on `name` is ignored
WHERE LOWER(e.name) = :name

// ✅ Sargable — index is used, input normalized in Java
WHERE e.name = :name
```

Normalize input in the service, not the database:
```java
public Post findBySlug(String slug) {
    return postRepository.findBySlug(slug.toLowerCase().trim());
}
```

## Gotchas
- Only works if stored data is already normalized (all lowercase on write, not just read)
- If stored data is mixed-case, you need a functional index:
  `CREATE INDEX idx_lower_name ON posts (LOWER(name));`
- Both JPQL and native queries are equally vulnerable

## Tags
#backend #database #performance #spring-boot #jpql

## Links
- [[Functional Indexes in PostgreSQL]]
- [[Service Layer Responsibilities]]
- [[The LEFT JOIN and WHERE Trap - Silent INNER JOIN]]