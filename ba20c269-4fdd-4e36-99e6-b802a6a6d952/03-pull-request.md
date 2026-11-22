### Purpose
Fixes the `AtomicIntegerArray` deserializer crashing with `IllegalStateException` (issue #1, upstream google/gson#3047).

### Description
A top-level JSON `null` and a `null` field were already handled (the adapter is `.nullSafe()`); I confirmed this with tests. The real crash was a `null` *element*, e.g. `[1,null,3]`, which leaked `IllegalStateException: Expected an int but was NULL` from `JsonReader.nextInt()`. The adapter now throws `JsonSyntaxException("null is not a valid AtomicIntegerArray element; at path $[1]")`, matching the existing `AtomicLongArray` behavior.

Suggestions considered:
- "Symmetric fix to #3038: make the AtomicIntegerArray deserializer handle JSON null (return null) instead of throwing IllegalStateException." Included. Top-level and field null already worked (tests added); null elements now give `JsonSyntaxException`.
- "Audit and fix the sibling atomic adapters (AtomicInteger, AtomicBoolean, AtomicLongArray) if they share the same defect." Included. All were already null-safe, so no change was needed.
- "Changes to Gson's public API or to the null-handling behavior of unrelated adapters." Excluded: out of scope for a maintenance-mode bug fix.

### Checklist
- [x] New code follows the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)\
  This is automatically checked by `mvn verify`, but can also be checked on its own using `mvn spotless:check`.\
  Style violations can be fixed using `mvn spotless:apply`; this can be done in a separate commit to verify that it did not cause undesired changes.\
  Result: `spotless:apply` was run and `mvn clean verify -pl gson -am` passed. `spotless:check` still reports a violation in the unmodified `ConcurrencyTest.java`, identically on the base; the changed files are clean.
- [x] If necessary, new public API validates arguments, for example rejects `null`. No new public API.
- [x] New public API has Javadoc. No new public API.
    - [x] Javadoc uses `@since $next-version$`  
      (`$next-version$` is a special placeholder which is automatically replaced during release). Not applicable.
- [x] If necessary, new unit tests have been added  
  - [x] Assertions in unit tests use [Truth](https://truth.dev/), see existing tests
  - [x] No JUnit 3 features are used (such as extending class `TestCase`)
  - [x] If this pull request fixes a bug, a new test was added for a situation which failed previously and is now fixed. The null-element test failed before the fix and passes now.
- [x] `mvn clean verify javadoc:jar` passes without errors. `mvn clean verify -pl gson -am` passed; `javadoc:jar` was not run.

### Screenshots
None; backend-only change with no rendered output.
