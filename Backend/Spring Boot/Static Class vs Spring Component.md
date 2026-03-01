
**One-liner:** If a class has no Spring dependencies, make it a static utility — don't inject it.

## Why It Matters
Marking a pure-logic class as `@Component` adds Spring overhead (proxy creation,
context scanning, bean lifecycle) for zero benefit. It also misleads readers
into thinking the class has dependencies it doesn't.

## Core Concept
```java
// ❌ Unnecessary Spring bean — no Spring deps used
@Component
public class EntityValidationUtils {
    public <T, ID> void validateAllFound(...) { /* pure Java */ }
}

// ✅ Static utility — honest, lightweight
public final class EntityValidationUtils {
    private EntityValidationUtils() {}

    public static <T, ID> void validateAllFound(...) { /* pure Java */ }
}
```

| Use `@Component`             | Use `static` utility               |
|------------------------------|------------------------------------|
| Needs `@Autowired` deps      | Pure Java logic only               |
| Needs to be mocked in tests  | Stateless, input → output          |
| Has lifecycle hooks          | No Spring context needed           |

## Gotchas
- Static utils are harder to mock — if you'll ever need to swap the implementation, use a bean
- `final` class = no subclassing (intentional for utils)
- `private` constructor = no accidental instantiation

## Tags
#backend #spring-boot #java #design-patterns #clean-code

## Links
- [[Spring Bean Lifecycle]] 
- [[Unit Testing - Mocking Static vs Instance Methods]] 
- [[Dependency Injection Principles]] 