```
@Component // ❌ unnecessary - no Spring dependencies public class EntityValidationUtils { public <T, ID> void validateAllFound(...) { // pure Java logic - doesn't need Spring at all } }


// As static utility public final class EntityValidationUtils { private EntityValidationUtils() {} // prevent instantiation public static <T, ID> void validateAllFound(...) { } }

```
if a class doesn't need any spring dependencies don't mark it as @Component etc, since this is pure java Util, static make sense.

`public final class EntityValidationUtils` making it final prevent sub-classes 
`private EntityValidationUtils() {}` making this private prevent creating objects