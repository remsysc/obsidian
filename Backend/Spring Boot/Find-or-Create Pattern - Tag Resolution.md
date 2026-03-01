**One-liner:** When an entity should be reused if it exists and created if it doesn't — use find-or-create.

## Why It Matters
Without this, you either duplicate records or make the caller responsible for pre-checking existence,
which breaks encapsulation. This pattern keeps that logic in one place.

## Core Concept
```java
private Set<Tag> resolveOrCreateTags(Set<String> tagNames) {
    return tagNames.stream()
        .map(name -> name.toLowerCase().trim())
        .map(name -> tagRepository.findByName(name)
            .orElseGet(() -> tagRepository.save(
                Tag.builder().name(name).build()
            )))
        .collect(Collectors.toSet());
}
```
```
Input name → normalize → findByName()
                              ↓
                        found?     → return existing
                        not found? → save new → return new
```

## Gotchas
- **N+1 problem**: runs one query per tag. Batch alternative:
```java
Set<Tag> existing = tagRepository.findAllByNameIn(normalizedNames);
// then create only the missing ones
```
- **Race condition**: two concurrent requests can both miss and both try to save → duplicate key.
  Mitigate with a unique constraint on `name` + retry logic or `ON CONFLICT DO NOTHING` (Postgres)
- Always normalize **before** querying, not after

## Tags
#backend #spring-boot #jpa #patterns #data-integrity

## Links
- [[Java Functional Interfaces and Method References]]