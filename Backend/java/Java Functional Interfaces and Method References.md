**One-liner:** Know which functional interface to reach for based on what goes in and what comes out.

## Why It Matters
Stream pipelines live on these four interfaces. Misidentifying them leads to
verbose, unreadable lambdas.

## Core Concept
```java
Function<T, R>   // T in → R out         .map(tag -> tag.getName())
Predicate<T>     // T in → boolean out   .filter(id -> !foundIds.contains(id))
Consumer<T>      // T in → nothing       .forEach(System.out::println)
Supplier<T>      // nothing → T out      .orElseGet(() -> new Tag())
```

Method references are shorthand lambdas — Java infers types:
```java
Tag::getId          ==   tag -> tag.getId()
String::toLowerCase ==   s -> s.toLowerCase()
tagRepo::save       ==   tag -> tagRepo.save(tag)
```

| Use method reference   | Use lambda                             |
|------------------------|----------------------------------------|
| Direct delegation      | Any logic beyond a single call         |
| Cleaner to read        | Multiple parameters need naming        |

## Gotchas
- `Supplier` with `orElseGet()` is lazy — the supplier only runs if the Optional is empty.
  `orElse()` always evaluates its argument. Prefer `orElseGet()` for expensive defaults.
- Don't force method references when they obscure intent

## Tags
#backend #java #functional-programming #streams #clean-code

## Links
- [[Stream API - map, filter, reduce patterns]]
- [[Optional - orElse vs orElseGet]]
- [[MapStruct Automatic Type Conversion by Signature Matching]]