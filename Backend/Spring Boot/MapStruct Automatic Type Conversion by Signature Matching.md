

**One-liner:** MapStruct finds converter methods by matching input/output types — you don't wire them manually.

## Why It Matters
Understanding this saves you hours of debugging "why isn't my custom mapper being called?"
MapStruct's type resolution is mechanical — if the signature matches, it applies automatically,
including across collections.

## Core Concept
MapStruct searches for a method that matches the type transformation needed:
```
Need:   Category  →  String
Finds:  default String categoryToString(Category category)
Result: applied automatically to every element in Set<Category>
```
```java
@Mapper(componentModel = "spring")
public interface PostMapper {

    PostDTO toDTO(Post post);

    // MapStruct auto-applies this to Set<Category> → Set<String>
    default String categoryToString(Category category) {
        return category.getName();
    }
}
```

You don't call `categoryToString` yourself — MapStruct wires it by type.

## Gotchas
- If two methods match the same type signature, MapStruct throws a compile error — be explicit
- Returning `null` from a converter method propagates nulls into collections — add null checks
- MapStruct runs at **compile time** — errors surface during build, not runtime

## Tags
#backend #java #mapstruct #mapping #spring-boot

## Links
- [[DTO Design — Create vs Update DTOs]]
- [[Java Functional Interfaces — Function, Predicate, Consumer, Supplier]]
- [[Static Class vs @Component]]