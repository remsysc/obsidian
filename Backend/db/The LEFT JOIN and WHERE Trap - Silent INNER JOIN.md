**One-liner:** A WHERE clause on a LEFT JOINed table silently converts it to an INNER JOIN.

## Why It Matters
You'll write a LEFT JOIN expecting to include rows with zero matches,
run it, and get results that look correct — but categories/authors with
0 posts are just gone. No error, no warning.

## Core Concept
```sql
-- ❌ WHERE kills the LEFT JOIN — rows with 0 posts disappear
SELECT c.name, COUNT(p.id)
FROM categories c
LEFT JOIN posts p ON c.id = p.category_id
WHERE p.status = 'PUBLISHED'
GROUP BY c.name

-- ✅ Filter inside ON — all categories returned, zeros show as 0
SELECT c.name, COUNT(p.id)
FROM categories c
LEFT JOIN posts p ON c.id = p.category_id AND p.status = 'PUBLISHED'
GROUP BY c.name
```

| Intent                          | Correct approach                              |
|---------------------------------|-----------------------------------------------|
| All rows, zeros included        | `LEFT JOIN ... ON col = :val`                 |
| Only rows with matches          | `INNER JOIN ... ON col = :val`                |
| Never do this                   | `LEFT JOIN ... WHERE joined_col = :val`       |

## Gotchas
- This applies equally to JPQL — it's not a SQL-only trap
- `COUNT(p.id)` correctly returns 0 for NULLs; `COUNT(*)` returns 1 — always use `COUNT(p.id)`
- If you want an INNER JOIN, just write INNER JOIN — don't hide it behind a WHERE

## Tags
#backend #database #sql #jpql #gotcha

## Links
- [[Aggregate Functions - COUNT ALL vs COUNT col]]
- [[DB Query Optimization - LOWER Rule]]