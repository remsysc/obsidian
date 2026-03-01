
**One-liner:** Know which functional interface to reach for based on what goes in and what comes out.

## Why It Matters
Stream pipelines live on these four interfaces. Misidentifying them leads to
verbose, unreadable lambdas. Recognizing the pattern makes you faster and cleaner.

## Core Concept
```java
Function<T, R>   // T in → R out         .map(tag -> tag.getName())
Predicate<T>     // T in → boolean out   .filter(id -> !foundIds.contains(id))
Consumer<T>      // T in → nothing       .forEach(System.out::println)
Supplier<T>      // nothing → T out      .orElseGet(() -> new Tag())
```

Method references are just shorthand lambdas — Java infers types:
```java
Tag::getId          ==   tag -> tag.getId()         // instance method ref
String::toLowerCase ==   s -> s.toLowerCase()       // instance method ref
tagRepo::save       ==   tag -> tagRepo.save(tag)   // instance method on captured object
```

When to use method reference vs lambda:
| Use method reference   | Use lambda                            |
|------------------------|---------------------------------------|
| Direct delegation      | Any logic beyond a single call        |
| Cleaner to read        | Multiple parameters need naming       |

## Gotchas
- `Supplier` is commonly used with `orElseGet()` — lazy evaluation means the
  supplier only runs if the Optional is empty (unlike `orElse()` which always evaluates)
- `Function.andThen()` and `Function.compose()` exist for chaining — learn them
- Don't force method references when they obscure intent

## Tags
#backend #java #functional-programming #streams #clean-code

## Links
- [[Stream API — map, filter, reduce patterns]]
- [[Optional — orElse vs orElseGet]]
- [[MapStruct Type Matching]]