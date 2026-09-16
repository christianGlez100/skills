---

name: kmp-unit-testing
description: Generate, review, and fix unit tests for Kotlin Multiplatform projects. Automatically inspect the project structure, source sets, targets, architecture, test dependencies, and Gradle configuration before generating tests. Perform a mandatory dependency pre-flight, prefer commonTest when possible, use kotlin.test, kotlinx-coroutines-test, Turbine, and manual Fakes when appropriate, and validate tests with the narrowest relevant Gradle test task.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# KMP Unit Testing Skill

## 1. Purpose

This skill is designed to generate, review, improve, and troubleshoot unit tests in **any Kotlin Multiplatform project**.

The skill MUST NOT assume:

* a specific project name
* a specific package
* a specific module name
* a specific architecture
* a specific dependency injection framework
* a specific test framework
* a specific set of KMP targets
* a specific source-set structure
* Astralia's architecture or dependencies

The skill MUST inspect the project before making decisions.

The project currently known as Astralia may be used as an architectural example, but all rules must remain generic and dynamically adaptable to other KMP projects.

---

# 2. Core Principles

Follow these principles in priority order:

1. Inspect before modifying.
2. Perform dependency pre-flight before generating tests.
3. Prefer the smallest compatible test scope.
4. Prefer `commonTest` when the tested behavior is platform-independent.
5. Prefer `kotlin.test` for common KMP assertions and test annotations.
6. Use `kotlinx-coroutines-test` for coroutine-based code.
7. Use Turbine when testing Flow-based observable behavior.
8. Prefer manual Fakes over mocking frameworks when interfaces make this possible.
9. Do not introduce a mocking framework unless it is actually necessary.
10. Never modify production code merely to make a test pass.
11. Never invent dependency versions.
12. Validate generated tests by running the narrowest applicable Gradle test task.
13. Distinguish dependency problems, test problems, and production-code problems.
14. Test observable behavior rather than implementation details.
15. Keep generated tests deterministic.
16. Adapt to the project's existing conventions whenever reasonable.

---

# 3. Project Discovery

Before generating or modifying tests, inspect the project.

Identify:

* Gradle root
* Gradle modules
* Kotlin Multiplatform modules
* `build.gradle.kts`
* `settings.gradle.kts`
* `gradle/libs.versions.toml`
* source sets
* KMP targets
* existing test source sets
* existing tests
* test dependencies
* dependency injection framework
* architecture
* package conventions
* existing testing conventions

Look for source sets such as:

```text
commonMain
commonTest

androidMain
androidHostTest
androidDeviceTest

iosMain
iosTest

jvmMain
jvmTest

jsMain
jsTest

wasmJsMain
wasmJsTest

desktopMain
desktopTest
```

Do not assume that all of them exist.

Determine which targets are actually configured.

---

# 4. Identify the Target Class

Before generating tests, determine:

* target file
* package
* module
* source set
* class type
* dependencies
* public API
* asynchronous behavior
* observable state
* exceptions
* platform-specific APIs

Classify the target when possible:

```text
ViewModel
Repository
Repository implementation
UseCase
DataSource
Service
Mapper
Validator
Parser
Utility
State holder
Controller
DI module
Platform implementation
Other
```

If classification is ambiguous, infer from the code instead of asking unnecessarily.

---

# 5. Mandatory Dependency Pre-Flight

This is a mandatory phase.

The skill MUST NOT generate test code before determining whether the required testing dependencies are available.

Inspect:

```text
gradle/libs.versions.toml
```

and the relevant:

```text
build.gradle.kts
```

files.

Also inspect existing tests to determine which testing libraries are already being used.

---

## 5.1 Determine Required Capabilities

Analyze the target class and determine which capabilities are required.

Examples:

### Basic unit testing

Required:

```text
kotlin.test
```

### Coroutine testing

Required when the target uses:

* suspend functions
* coroutine builders
* `viewModelScope`
* `CoroutineScope`
* dispatchers
* asynchronous coroutine behavior

Preferred:

```text
org.jetbrains.kotlinx:kotlinx-coroutines-test
```

Use:

```kotlin
runTest
```

when appropriate.

### Flow testing

Required/recommended when testing:

* Flow
* StateFlow
* SharedFlow
* cold flows
* hot flows
* emitted state transitions

Preferred:

```text
app.cash.turbine:turbine
```

Use APIs such as:

```kotlin
test
awaitItem
awaitComplete
awaitError
ensureAllEventsConsumed
```

when appropriate.

### Dependency injection testing

If the test specifically validates the DI graph, inspect whether the project already uses:

```text
Koin
Dagger
Hilt
Kodein
Manual DI
Other
```

Use existing project tooling whenever possible.

Do not introduce a DI testing library merely to test a class that can be instantiated directly.

### Mocking

Do not automatically require:

```text
Mockito
MockK
Mockative
Mokkery
```

First determine whether a manual Fake is sufficient.

---

# 6. Dependency Matrix

Create an internal dependency matrix before generating tests.

Example:

| Requirement                 | Dependency/strategy       | Installed | Required |
| --------------------------- | ------------------------- | --------: | -------: |
| Test annotations/assertions | `kotlin.test`             |         ✅ |        ✅ |
| Coroutine testing           | `kotlinx-coroutines-test` |         ❌ |        ✅ |
| Flow testing                | Turbine                   |         ❌ |        ✅ |
| DI testing                  | Existing DI test library  |         ✅ |        ❌ |
| Mocking                     | Manual Fake               |       N/A |        ✅ |

The exact dependency names must be discovered from the project and official/current documentation when necessary.

---

# 7. Missing Dependency Policy

If a required dependency is missing, STOP test generation.

Report:

```text
KMP Unit Test Pre-flight

Target:
<target>

Missing dependency:
<dependency>

Gradle coordinate:
<coordinate>

Required in:
<source set>

Why:
<reason>

Impact:
<what cannot be tested conveniently without it>
```

Example:

```text
⚠️ KMP Unit Test Pre-flight

Target: UserViewModel

❌ Missing: kotlinx-coroutines-test

Required in:
commonTest

Why:
The ViewModel launches coroutine work and requires deterministic coroutine
execution using runTest.

❌ Missing: Turbine

Required in:
commonTest

Why:
The ViewModel exposes StateFlow and the tests need to verify emitted states.

No mocking library is required because UserRepository is an interface and
can be replaced with a manual Fake.
```

Then offer these actions:

```text
1. Add missing dependencies and continue
2. Only report the missing dependencies
3. Cancel
```

Do not silently modify dependencies.

If the environment allows repository modification, only modify dependencies after explicit approval.

---

# 8. Dependency Version Rules

Never invent versions.

Preferred order:

1. Existing project version.
2. Version already defined in the project's version catalog.
3. Compatible version documented by the library.
4. Current official release after verification.

Do not unnecessarily upgrade unrelated dependencies.

Do not change Kotlin, Gradle, AGP, Compose, or other project versions merely to add a test dependency.

If compatibility is uncertain, report it instead of guessing.

---

# 9. Source Set Selection

Choose the smallest source set that can correctly test the behavior.

Preferred order:

```text
commonTest
↓
platform-specific test source set
```

Use `commonTest` when:

* production code is common
* dependencies are KMP-compatible
* behavior is platform-independent
* no platform API is required

Use Android-specific tests when:

* Android APIs are required
* Android lifecycle behavior must be tested
* Android-only implementation is under test

Use iOS-specific tests when:

* the implementation uses iOS APIs
* behavior cannot be tested in common code

Use JS/Wasm-specific tests when:

* browser/platform APIs are involved
* the tested implementation is target-specific

Never force platform-independent logic into platform-specific tests.

---

# 10. Test Strategy by Class Type

## 10.1 ViewModel

For a ViewModel, identify:

* initial state
* public actions
* state transitions
* successful result
* empty result
* failure
* exception messages
* parameters passed to dependencies
* repeated invocation behavior
* cancellation behavior when relevant

For coroutine-based ViewModels, prefer:

```kotlin
runTest
```

For Flow/StateFlow:

```kotlin
Turbine
```

Avoid testing private implementation details.

Example strategy:

```text
Given repository returns data
When ViewModel action executes
Then state becomes Success(data)
```

Also test:

```text
Loading
Success
Empty result
Error
Unknown error
```

when those states exist in the production contract.

---

# 11. Repository Testing

If a repository depends on an interface:

```kotlin
interface UserDataSource
```

prefer a manual Fake.

If the repository depends directly on a concrete external implementation such as:

```kotlin
SupabaseClient
```

or another external SDK:

do not automatically introduce a mocking framework.

First determine whether:

1. the class is suitable for a unit test
2. the dependency should be abstracted
3. the test should be an integration test

Do not change production architecture solely to satisfy the generated test.

If abstraction is clearly beneficial, report it separately as a recommendation.

---

# 12. DataSource Testing

Determine whether the DataSource is:

* pure logic
* local storage
* database access
* network access
* external SDK integration

Pure logic can generally be unit tested.

Network/database/external SDK behavior may require integration testing.

Do not pretend an integration test is a unit test.

---

# 13. UseCase Testing

For a UseCase:

* use manual Fakes for dependencies
* test successful behavior
* test invalid inputs
* test dependency failures
* test transformation/business rules
* test returned values

Avoid testing private implementation details.

---

# 14. Mapper Testing

Mappers should normally be simple unit tests.

Test:

* all mapped fields
* nullable fields
* default values
* enum conversions
* nested objects
* malformed/edge cases when applicable

No mocking framework should normally be necessary.

---

# 15. Manual Fakes

Manual Fakes are the default test-double strategy.

Prefer:

```kotlin
class FakeUserRepository : UserRepository {

    var result: List<User> = emptyList()
    var exception: Throwable? = null

    override suspend fun getUsers(): List<User> {
        exception?.let { throw it }
        return result
    }
}
```

Use recording properties when parameter verification is necessary:

```kotlin
var receivedUserId: String? = null
```

Keep Fakes inside the test source set.

Do not place test-only classes in production source sets.

---

# 16. Mocking Framework Policy

A mocking library should only be introduced when:

* the dependency cannot reasonably be faked
* the project already standardizes on a mocking framework
* the test requires behavior that is impractical with a Fake
* the user explicitly requests mocking

Before adding a mocking framework, verify:

* KMP support
* target support
* Kotlin compatibility
* Gradle compatibility
* source-set compatibility

Never add Mockito/MockK/etc. by default to a KMP `commonTest`.

---

# 17. Coroutine Testing

Never use:

```kotlin
Thread.sleep(...)
```

for coroutine synchronization.

Avoid arbitrary delays.

Prefer:

```kotlin
runTest {
    ...
}
```

Use virtual time when applicable.

If production code injects a dispatcher, use the project's existing dispatcher abstraction.

Do not introduce unnecessary dispatcher abstractions solely for testing.

---

# 18. Flow Testing

For Flow-based APIs, prefer Turbine when it improves readability and determinism.

Example:

```kotlin
@Test
fun `state emits loading and success`() = runTest {
    viewModel.state.test {
        assertEquals(Loading, awaitItem())

        viewModel.load()

        assertEquals(Success(expected), awaitItem())

        cancelAndIgnoreRemainingEvents()
    }
}
```

Adapt the exact test to the project's state model.

Do not assume the first StateFlow value is `Loading`; inspect the production code.

---

# 19. StateFlow Testing

When testing StateFlow:

1. Inspect its initial value.
2. Determine whether the state is eagerly or lazily initialized.
3. Trigger the public operation.
4. Assert observable state transitions.
5. Avoid asserting internal MutableStateFlow implementation.

If only the final state matters, do not unnecessarily assert every intermediate emission.

---

# 20. Exception Testing

Test exceptions through observable behavior.

If production code converts exceptions into an error state:

```kotlin
catch (e: Exception) {
    state = Error(e.message ?: "Unknown error")
}
```

test both:

```text
exception with message
exception without message
```

when that behavior is part of the contract.

Do not test `printStackTrace()` or logging unless logging itself is explicitly part of the requirement.

---

# 21. Test Naming

Follow existing project conventions.

If no convention exists, prefer descriptive behavior-oriented names.

Examples:

```text
loadUsers emits success when repository returns users
loadUsers emits error when repository throws exception
loadUsers returns empty state when repository returns no users
```

Kotlin backtick names are acceptable:

```kotlin
@Test
fun `load users emits success when repository returns users`() {
}
```

---

# 22. Test Structure

Prefer Arrange / Act / Assert or Given / When / Then.

Example:

```kotlin
// Arrange
fakeRepository.result = expectedUsers
val viewModel = UserViewModel(fakeRepository)

// Act
viewModel.loadUsers()

// Assert
...
```

Keep each test focused on one behavior.

Avoid giant tests covering unrelated behaviors.

---

# 23. Test Data

Use readable test fixtures.

Prefer:

```kotlin
private val user = User(
    id = "1",
    name = "John"
)
```

instead of repeatedly constructing large objects inside tests.

Avoid unrealistic test data unless testing an edge case.

Use builders or fixture helpers only when they materially improve readability.

Do not introduce a test-data framework unnecessarily.

---

# 24. Architecture Awareness

The skill must adapt to the project's architecture.

Potential architectures include:

```text
MVVM
MVI
Clean Architecture
Redux
VIPER
Repository pattern
UseCase pattern
Unidirectional Data Flow
Custom architecture
```

Do not impose Clean Architecture or any other pattern.

Tests should follow the production architecture rather than redesign it.

---

# 25. Dependency Injection

If a class can be directly instantiated:

```kotlin
val viewModel = UserViewModel(fakeRepository)
```

prefer this for unit tests.

Do not start the complete DI container unless testing the DI configuration itself.

Use DI integration tests for:

```text
module registration
bindings
qualifiers
scopes
dependency resolution
```

Use unit tests for business behavior.

---

# 26. Koin

If the project uses Koin:

Use `koin-test` only when testing Koin configuration or dependency resolution.

Do not require Koin to test a class whose dependencies can be supplied directly.

Example:

```kotlin
val viewModel = UserViewModel(fakeRepository)
```

is preferable to starting Koin for an ordinary ViewModel unit test.

---

# 27. Existing Tests

Before creating a new test:

1. Inspect nearby tests.
2. Reuse existing fixtures when appropriate.
3. Follow naming conventions.
4. Follow import conventions.
5. Follow source-set conventions.
6. Avoid creating duplicate utilities.

Never overwrite an existing test without understanding its purpose.

---

# 28. Test File Location

Determine the production class source-set path.

For common production code, prefer:

```text
<module>/src/commonTest/
```

Mirror the production package.

Example:

```text
commonMain:
com.example.feature.UserViewModel

commonTest:
com.example.feature.UserViewModelTest
```

Do not create tests in arbitrary folders.

---

# 29. Test Execution

After generating tests, identify the narrowest relevant Gradle task.

Examples may include:

```bash
./gradlew :shared:test
```

or:

```bash
./gradlew :shared:wasmJsTest
./gradlew :shared:jsTest
./gradlew :shared:iosSimulatorArm64Test
./gradlew :shared:testAndroidHostTest
```

These are examples only.

The skill MUST inspect the actual project's configured tasks before selecting the command.

Prefer the smallest task that validates the changed test.

---

# 30. Validation Workflow

After test generation:

```text
Generate
   ↓
Compile
   ↓
Run targeted test
   ↓
Analyze failure
   ↓
Fix test if appropriate
   ↓
Run again
```

If compilation fails, determine whether the cause is:

```text
1. Missing dependency
2. Wrong import
3. Wrong source set
4. Incorrect test code
5. Production-code compilation error
6. Platform incompatibility
7. Gradle configuration problem
```

Do not blindly modify production code.

---

# 31. Failure Classification

Always distinguish:

### Dependency failure

Example:

```text
Could not resolve kotlinx-coroutines-test
```

Report as dependency/configuration issue.

### Test implementation failure

Example:

```text
Expected Success but was Loading
```

Inspect the test and production behavior.

### Production failure

Example:

```text
NullPointerException in production class
```

Report that the production implementation appears to fail under the tested scenario.

Do not automatically "fix" production code.

### Platform failure

Example:

```text
API available on Android but not commonTest
```

Move the test to the correct platform source set or adapt the strategy.

---

# 32. Production Code Modification Policy

The skill MUST NOT modify production code simply because:

* a test is difficult to write
* a mock would be convenient
* a dependency is hard to fake
* a private implementation detail is inaccessible

Production code may be changed only when:

1. the user explicitly requests refactoring
2. the current architecture genuinely prevents reasonable testing
3. the modification is justified and minimal
4. the user is informed

When possible, report the refactoring as a recommendation instead of performing it automatically.

---

# 33. Dependency Modification Policy

If dependencies are missing:

Do not silently edit:

```text
libs.versions.toml
build.gradle.kts
settings.gradle.kts
```

Ask for approval unless the user explicitly authorized automatic dependency changes.

When adding a dependency:

1. Add the version to the version catalog when the project uses one.
2. Add the library alias.
3. Add it to the smallest appropriate source set.
4. Avoid adding it globally.
5. Preserve existing dependency style.
6. Do not upgrade unrelated libraries.
7. Verify target compatibility.

---

# 34. KMP Compatibility

Every testing dependency must be evaluated against the project's actual targets.

For example:

```text
commonTest
├── Android
├── iOS
├── JS
└── Wasm
```

A dependency that works for JVM/Android does not automatically mean it is suitable for:

```text
iOS
JS
Wasm
```

If compatibility is incomplete:

```text
⚠️ Compatibility concern

Dependency:
<dependency>

Works with:
<targets>

Does not support:
<targets>

Recommendation:
Use it only in <source set> or choose another strategy.
```

Never assume JVM-only testing libraries are appropriate for KMP commonTest.

---

# 35. Generic Project Adaptation

The skill must dynamically adapt to projects such as:

```text
Project A
shared/commonMain
shared/commonTest
Android/iOS

Project B
composeApp/commonMain
composeApp/commonTest
Android/iOS/Desktop

Project C
core/commonMain
core/commonTest
Android/iOS/JS/Wasm

Project D
shared/commonMain
shared/commonTest
Android/iOS
with platform-specific implementations
```

The skill must discover the actual structure.

---

# 36. Astralia Compatibility Profile

Astralia can be used as a reference implementation.

Known characteristics include:

```text
Kotlin Multiplatform
Compose Multiplatform
Android
iOS
JS
Wasm
commonMain
commonTest
Koin
Supabase
ViewModels
StateFlow
Repository pattern
DataSource pattern
```

However, these characteristics are NOT requirements of this Skill.

When operating on Astralia, the skill may use the detected architecture naturally.

When operating on another project, ignore this profile and rediscover the project.

---

# 37. Recommended Test Priority

When asked to generate tests for a class, prioritize:

### Priority 1

Public behavior and success path.

### Priority 2

Error behavior.

### Priority 3

Empty/null/edge cases.

### Priority 4

Parameter propagation.

### Priority 5

State transitions.

### Priority 6

Cancellation/concurrency behavior when relevant.

Avoid writing tests merely to increase line coverage.

---

# 38. Coverage Philosophy

Coverage is useful but is not the primary objective.

Prefer:

```text
behavioral confidence
```

over:

```text
maximum line coverage
```

Do not generate meaningless tests simply to execute every branch.

Prioritize business-critical behavior.

---

# 39. Anti-Patterns

Avoid:

```text
Thread.sleep()
```

```text
runBlocking
```

when `runTest` is appropriate.

Avoid:

```text
mock everything
```

Avoid starting a complete DI graph for simple unit tests.

Avoid testing private functions directly.

Avoid testing implementation details.

Avoid asserting exact internal coroutine structure.

Avoid platform-specific tests when commonTest is sufficient.

Avoid introducing dependencies without checking compatibility.

Avoid inventing dependency versions.

Avoid changing production code to accommodate poor tests.

Avoid excessive test duplication.

---

# 40. Mandatory Workflow

Every test-generation request MUST follow this workflow:

```text
1. Parse user request
        ↓
2. Locate target class
        ↓
3. Inspect project structure
        ↓
4. Inspect source sets
        ↓
5. Inspect build configuration
        ↓
6. Inspect version catalog
        ↓
7. Inspect existing tests
        ↓
8. Analyze target class
        ↓
9. Determine required testing capabilities
        ↓
10. Run dependency pre-flight
        ↓
11. Detect compatibility issues
        ↓
12. STOP if required dependencies are missing
        ↓
13. Build test strategy
        ↓
14. Choose source set
        ↓
15. Choose Fake/Mock/Stub strategy
        ↓
16. Generate tests
        ↓
17. Run targeted Gradle test
        ↓
18. Diagnose failures
        ↓
19. Fix test implementation if necessary
        ↓
20. Re-run tests
        ↓
21. Report result
```

Step 10 is mandatory.

Step 12 must not be skipped.

---

# 41. Pre-Flight Output Format

Use this format before generating tests:

```text
## KMP Unit Test Pre-flight

Target:
<file/class>

Module:
<module>

Source set:
<source set>

Test strategy:
<strategy>

Dependencies:

✅ <dependency>
   Reason: <reason>

❌ <dependency>
   Reason: <reason>
   Required in: <source set>

ℹ️ <dependency>
   Available but not required

Test doubles:

✅ Manual Fake
❌ Mocking framework not required

Compatibility:

✅ commonTest compatible
⚠️ <compatibility issue>

Result:

✅ Pre-flight passed
```

or:

```text
⛔ Pre-flight blocked

Missing dependencies:
- ...
- ...

No test code will be generated until the dependency requirements are resolved.
```

---

# 42. Final Output Format

After successful generation, report:

```text
## Unit Tests Generated

Target:
<target>

Test file:
<path>

Source set:
<source set>

Tests added:
- <test>
- <test>
- <test>

Dependencies:
- <dependency>

Test doubles:
- <fake>

Validation:
✅ Compilation passed
✅ Tests passed

Command:
<actual Gradle command>
```

If validation fails:

```text
## Unit Test Result

Generated:
<yes/no>

Compilation:
✅/❌

Tests:
✅/❌

Failure:
<description>

Classification:
<dependency | test | production | platform | Gradle>

Recommended next action:
<action>
```

---

# 43. Minimalism Rule

Do not generate more infrastructure than necessary.

For a simple ViewModel:

```text
ViewModel
+
Fake Repository
+
kotlin.test
+
coroutines-test
+
Turbine
```

may be sufficient.

Do not automatically add:

```text
Mockito
MockK
fixture libraries
assertion libraries
DI test frameworks
integration-test infrastructure
```

unless the project or test actually requires them.

---

# 44. Security and Reliability

Never expose:

* API keys
* Supabase keys
* tokens
* credentials
* secrets

inside generated tests.

Use:

```text
fake values
test fixtures
environment variables
test configuration
```

when required.

Never copy production credentials into tests.

---

# 45. Definition of Done

A unit-test task is complete only when:

* target class was inspected
* project structure was inspected
* source set was selected intentionally
* dependencies were checked
* compatibility was evaluated
* missing dependencies were reported/resolved
* appropriate test doubles were selected
* tests were generated
* tests compile
* targeted tests execute
* failures were classified
* no unnecessary production changes were introduced

## The skill should prefer a smaller number of meaningful, passing tests over a large number of superficial tests.

## End of Skill
