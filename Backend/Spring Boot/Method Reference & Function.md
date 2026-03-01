```
// Method references
Tag::getId  ==  tag -> tag.getId()

// Functional interfaces
Function<T,R>   // takes T, returns R
Predicate<T>    // takes T, returns boolean
Consumer<T>     // takes T, returns nothing
Supplier<T>     // takes nothing, returns T

// Lambda without explicit type - Java infers it
.filter(id -> !foundIds.contains(id))
.map(tag -> tag.getId())
```

[[MapStruct Type Matching]]