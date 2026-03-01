`-- ❌ WHERE kills LEFT JOIN — categories with 0 posts disappear silently LEFT JOIN c.posts p WHERE p.status = 'PUBLISHED' -- ✅ Condition inside ON — all categories returned, zeros included LEFT JOIN c.posts p ON p.status = 'PUBLISHED'`

## The JOIN Rule — Most Important Takeaway

| Intent                         | Correct Query                            |
| ------------------------------ | ---------------------------------------- |
| All categories, zeros included | `LEFT JOIN ... ON p.status = :status`    |
| Only categories with posts     | `INNER JOIN ... ON p.status = :status`   |
| Never do this                  | `LEFT JOIN ... WHERE p.status = :status` |
## The One Rule to Carry Forward

> **Never filter a `LEFT JOIN` with a `WHERE` on the joined table's column. It silently becomes an `INNER JOIN` — no error, no warning, just missing data.** 🎯

[[DB OPTIMIZATION QUERIES]]