

**One-liner:** When an entity should be reused if it exists and created if it doesn't — use find-or-create.

## Why It Matters
Without this, you either duplicate records (two tags both named "java")
or make the caller responsible for pre-checking existence — which breaks encapsulation.
This pattern keeps that logic in one place.

## Core Concept
```java
private Set<Tag> resolveOrCreateTags(Set<String> tagNames) {
    return tagNames.stream()
        .map(name -> name.toLowerCase().trim())       // normalize first
        .map(name -> tagRepository.findByName(name)
            .orElseGet(() -> tagRepository.save(      // find OR create
                Tag.builder().name(name).build()
            )))
        .collect(Collectors.toSet());
}
```

Flow:
```
Input name → normalize → findByName()
                              ↓
                        found? → return existing
                        not found? → save new → return new
```

## Gotchas
- **N+1 problem**: this runs one query per tag — fine for small sets,
  problematic at scale. Batch alternative:
```java
  // fetch all existing in one query
  Set<Tag> existing = tagRepository.findAllByNameIn(normalizedNames);
  // create only the missing ones
```
- **Race condition**: two concurrent requests can both "not find" the tag
  and both try to save → duplicate key exception. Mitigate with a
  unique constraint on `name` + retry logic or `ON CONFLICT` (Postgres)
- Always normalize (lowercase + trim) **before** querying — not after

## Tags
#backend #spring-boot #jpa #patterns #data-integrity

## Links
- [[Batch Repository Methods — findAllByXIn()]]
- [[Unique Constraints + Conflict Handling in JPA]]
- [[Java Functional Interfaces & Method References]]