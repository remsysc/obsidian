**One-liner:** `COUNT(*)` counts every row; `COUNT(col)` skips NULLs — they are not interchangeable.

## Why It Matters
Using the wrong one in a LEFT JOIN gives you silently wrong numbers.
`COUNT(*)` on a LEFT JOIN returns 1 for unmatched rows instead of 0.

## Core Concept
```sql
COUNT(*)      -- counts all rows, including NULLs
COUNT(p.id)   -- counts only non-NULL values in p.id
```

In a LEFT JOIN context:
```sql
-- ✅ Correct: returns 0 for categories with no posts
SELECT c.name, COUNT(p.id)
FROM categories c
LEFT JOIN posts p ON c.id = p.category_id
GROUP BY c.name

-- ❌ Wrong: returns 1 for categories with no posts
SELECT c.name, COUNT(*)
FROM categories c
LEFT JOIN posts p ON c.id = p.category_id
GROUP BY c.name
```

## Gotchas
- Always use `COUNT(col)` when aggregating over a LEFT JOINed table
- `COUNT(DISTINCT col)` also ignores NULLs — consistent with `COUNT(col)`
- This applies equally in JPQL, not just native SQL

## Tags
#backend #database #sql #jpql #gotcha

## Links
- [[The LEFT JOIN and WHERE Trap - Silent INNER JOIN]]
- [[DB Query Optimization - LOWER Rule]]