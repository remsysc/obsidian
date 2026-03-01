**One-liner:** Never let the client control fields the system should own.

## Why It Matters
If you let users send `createdAt` or `readingTime` in the request body,
you lose data integrity. A malicious or buggy client can forge timestamps
or bypass business logic entirely.

## Core Concept
| Owner  | Fields                              |
|--------|-------------------------------------|
| User   | title, content, categories, tags    |
| System | createdAt, updatedAt, readingTime   |
```java
@PrePersist
void onCreate() {
    this.createdAt = Instant.now();
    this.readingTime = calculateReadingTime(this.content);
}

@PreUpdate
void onUpdate() {
    this.updatedAt = Instant.now();
    this.readingTime = calculateReadingTime(this.content);
}
```

## Gotchas
- Shared DTOs for create + update let system fields leak into the request body — use separate DTOs or `@JsonIgnore`
- `@PrePersist` / `@PreUpdate` only fire through JPA — bulk JPQL updates bypass them silently
- Never trust `readingTime` sent from the client, even if it looks harmless

## Tags
#backend #spring-boot #jpa #api-design #data-integrity

## Links
- [[API Design - Business Requirements Over Industry Standards]]