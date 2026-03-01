

#spring-boot #java #backend #performance #rest-api

---

## What Is Pagination?

Instead of loading **every record** from the database at once, pagination returns a **controlled slice** of data with metadata telling the client how to navigate the rest.

```json
{
  "content": [ ...20 posts... ],
  "pageNumber": 0,
  "pageSize": 20,
  "totalElements": 143,
  "totalPages": 8,
  "first": true,
  "last": false
}
```

---

## When To Use It

> **Any endpoint returning a user-generated or unbounded collection should be paginated by default.**

|Collection Type|Paginate?|
|---|---|
|Posts, comments, users|✅ Always|
|Search results|✅ Always|
|Tags, categories (user-generated)|✅ Yes|
|Admin-controlled, naturally small lists|⚠️ Recommended|
|Single resource `GET /posts/{id}`|❌ Never|

### You're Late When You See...

- `OutOfMemoryError` on list endpoints
- Response times > 2-3 seconds on GET
- Database CPU spikes on list queries
- Frontend freezing while rendering

---

## Core Imports

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;   // manual Page construction
import org.springframework.data.domain.Pageable;
import org.springframework.data.web.PageableDefault;
import org.springframework.data.domain.Sort;
```

> ⚠️ **Never import** `java.awt.print.Pageable` — wrong library, same name.

---

## Implementation — All Four Layers

### 1. Repository

Use the **Two-Query Pattern** to avoid in-memory pagination.

```java
// Query 1: paginate IDs only at DB level
@Query(value = """
    SELECT p.id FROM Post p
    WHERE (:categoryId IS NULL OR EXISTS(
        SELECT 1 FROM p.categories c WHERE c.id = :categoryId))
    AND (:tagId IS NULL OR EXISTS(
        SELECT 1 FROM p.tags t WHERE t.id = :tagId))
    """,
    countQuery = """
    SELECT COUNT(p) FROM Post p
    WHERE (:categoryId IS NULL OR EXISTS(
        SELECT 1 FROM p.categories c WHERE c.id = :categoryId))
    AND (:tagId IS NULL OR EXISTS(
        SELECT 1 FROM p.tags t WHERE t.id = :tagId))
    """)
Page<UUID> findPostIds(
    @Param("categoryId") UUID categoryId,
    @Param("tagId") UUID tagId,
    Pageable pageable
);

// Query 2: fetch full data for known IDs only
@Query("""
    SELECT DISTINCT p FROM Post p
    JOIN FETCH p.author
    LEFT JOIN FETCH p.categories
    LEFT JOIN FETCH p.tags
    WHERE p.id IN :ids
""")
List<Post> findAllByIdIn(@Param("ids") List<UUID> ids);
```

### 2. Service Interface

```java
Page<PostResponse> getAllPosts(UUID categoryId, UUID tagId, Pageable pageable);
```

### 3. Service Implementation

```java
public Page<PostResponse> getAllPosts(UUID categoryId, UUID tagId, Pageable pageable) {

    // Step 1: DB-level pagination on IDs (fast)
    Page<UUID> idPage = postRepository.findPostIds(categoryId, tagId, pageable);

    // Step 2: hydrate full entities for just this page's IDs
    List<Post> posts = postRepository.findAllByIdIn(idPage.getContent());

    // Step 3: re-wrap into Page to preserve metadata
    return new PageImpl<>(
        postMapper.toResponses(posts),
        pageable,
        idPage.getTotalElements()
    );
}
```

### 4. Controller

```java
@GetMapping
public ResponseEntity<ApiResponse<PageResponse<PostResponse>>> getAllPosts(
    @RequestParam(required = false) UUID categoryId,
    @RequestParam(required = false) UUID tagId,
    @PageableDefault(size = 20, sort = "createdAt",
        direction = Sort.Direction.DESC) Pageable pageable) {

    Page<PostResponse> posts = postService.getAllPosts(categoryId, tagId, pageable);
    return ResponseEntity.ok(
        ApiResponse.success("Retrieved all posts", PageResponse.from(posts))
    );
}
```

---

## Custom PageResponse DTO

Wrap Spring's raw `Page<T>` to avoid leaking framework internals to clients.

```java
@Data
@Builder
public class PageResponse<T> {

    private List<T> content;
    private int pageNumber;
    private int pageSize;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;

    public static <T> PageResponse<T> from(Page<T> page) {
        return PageResponse.<T>builder()
                .content(page.getContent())
                .pageNumber(page.getNumber())
                .pageSize(page.getSize())
                .totalElements(page.getTotalElements())
                .totalPages(page.getTotalPages())
                .first(page.isFirst())
                .last(page.isLast())
                .build();
    }
}
```

### Why Not Return Raw `Page<T>`?

| Concern       | Raw `Page<T>`                     | Custom `PageResponse<T>` |
| ------------- | --------------------------------- | ------------------------ |
| API stability | Breaks if Spring internals change | You control the contract |
| Clarity       | Duplicate/confusing fields        | Only what client needs   |

---

## The Two-Query Pattern — Deep Dive

### Why `JOIN FETCH` + `Pageable` Breaks

When you join posts with categories, one post produces **multiple rows**:

```
post_id  | title    | category
---------|----------|----------
uuid-1   | Post A   | tech        ← same post
uuid-1   | Post A   | gaming      ← same post
uuid-2   | Post B   | gaming
```

Hibernate can't apply `LIMIT 20` safely because 20 rows ≠ 20 posts. So it fetches **everything** and paginates in Java memory.

```
// What you want
PostgreSQL → 20 rows → done

// What actually happens with JOIN FETCH + Pageable
PostgreSQL → ALL rows → Hibernate cuts to 20 in memory ❌
```

### Why `EXISTS` Doesn't Need `DISTINCT`

`JOIN` → one row per relationship match → duplicates → need `DISTINCT`

`EXISTS` → just checks "does a match exist?" → one row per post → no duplicates

### Two-Query Flow

```
Request
  │
  ├─ Query 1: SELECT p.id ... LIMIT 20 OFFSET 0
  │           → PostgreSQL returns [uuid1, uuid2, ... uuid20] + total count
  │
  ├─ Query 2: SELECT p.* JOIN FETCH ... WHERE p.id IN (uuid1...uuid20)
  │           → PostgreSQL returns full data for exactly 20 posts
  │
  └─ PageImpl wraps List<PostResponse> + metadata → Client
```

### Performance Comparison

|Concern|Single Query|Two-Query Pattern|
|---|---|---|
|DB rows fetched|All rows|Page size only|
|Pagination location|Java memory|Database|
|`LazyInitializationException` risk|Yes|No|
|Performance at scale|Degrades|Stays constant|
|Query count|1 (expensive)|2 (both cheap)|
> **Two cheap queries always beats one expensive query.**

---

## Common Errors & Fixes

### `InvalidDataAccessResourceUsageException`

```
ERROR: for SELECT DISTINCT, ORDER BY expressions must appear in select list
```

**Cause:** Using `DISTINCT` with `ORDER BY` on a column not in the `SELECT`.  
**Fix:** Remove `DISTINCT` — use `EXISTS` subqueries which don't produce duplicates.

### `LazyInitializationException`

```
could not initialize proxy - no Session
```

**Cause:** Accessing a `FetchType.LAZY` collection outside a transaction (especially with `open-in-view=false`).  
**Fix:** Add the missing association to your `JOIN FETCH` clause.

### `HHH90003004` Hibernate Warning

```
firstResult/maxResults specified with collection fetch; applying in memory
```

**Cause:** Using `JOIN FETCH` with `Pageable`.  
**Fix:** Use the two-query pattern.

---

## Client Usage

```
GET /api/v1/posts                          → page 0, size 20, default sort
GET /api/v1/posts?page=2                   → page 2
GET /api/v1/posts?page=0&size=10           → 10 per page
GET /api/v1/posts?sort=createdAt,desc      → custom sort
GET /api/v1/posts?categoryId=uuid&page=1   → filtered + paginated
```

---

## Related Notes

- [[Spring Boot Repository Layer]]
- [[JPA Fetch Types - LAZY vs EAGER]]
- [[N+1 Query Problem]]
- [[REST API Design Conventions]]
- [[Spring Security Config]]