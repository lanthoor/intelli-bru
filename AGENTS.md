# AGENTS.md - Guidelines for AI Coding Agents

This document provides essential information for AI agents working on the intelli-bru codebase.

## Project Overview

**intelli-bru** is a JetBrains IntelliJ Platform Plugin written in Kotlin using Jetpack Compose UI with Jewel bridge.

- **Language:** Kotlin 2.1.20 | **JVM:** 21 | **Gradle:** 9.0.0 (Kotlin DSL)
- **UI Framework:** Jetpack Compose with Jewel (org.jetbrains.jewel)
- **Target IDE:** IntelliJ IDEA 2025.2.4+ (since build 252.25557)
- **Group ID:** `in.co.spendly`

## Development Workflow

Follow this iterative workflow for each phase/sub-phase. **Each phase or sub-phase = one commit.**

```
┌─────────────────────────────────────────────────────────────┐
│                    ITERATIVE WORKFLOW                       │
│                                                             │
│   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
│   │Implement│──►│  Test   │──►│ Verify  │──►│ Commit  │   │
│   └─────────┘   └─────────┘   └─────────┘   └─────────┘   │
│        │                            │             │        │
│        │                            │             ▼        │
│        │                            │      Next Phase      │
│        ▼                            ▼                      │
│   If tests fail ◄──────────── Fix issues                   │
└─────────────────────────────────────────────────────────────┘
```

### Step 1: Implement Feature/Fix
- Write production code for the phase/sub-phase
- Follow code style guidelines below
- Update `plugin.xml` if adding extensions/actions

### Step 2: Add/Update Tests
- Add unit tests for new functionality
- Update existing tests if behavior changed
- Place tests in `src/test/kotlin/` mirroring main structure

### Step 3: Verify All Checks Pass
```bash
./gradlew check                    # Run all verification (includes test)
./gradlew verifyPlugin             # Verify plugin compatibility
```
**Do NOT proceed to commit if any check fails.**

### Step 4: Commit
```bash
git add -A
git commit -m "type(scope): description"
```

**Conventional Commit Format:** `type(scope): description`

| Type | Use When |
|------|----------|
| `feat` | Adding new functionality |
| `fix` | Fixing a bug |
| `refactor` | Code changes that neither fix bugs nor add features |
| `test` | Adding or updating tests |
| `docs` | Documentation changes |
| `chore` | Build, config, or tooling changes |
| `style` | Code style/formatting (no logic change) |

**Scope:** Use phase number or component name
- `feat(phase-1.1): add Grammar-Kit plugin configuration`
- `feat(bru-lexer): implement JFlex lexer specification`
- `fix(cli): handle timeout on slow requests`
- `test(parser): add tests for multiline values`
- `docs(readme): add installation instructions`
- `chore(build): update Kotlin to 2.1.20`

### Step 5: Move to Next Phase
Only after successful commit, proceed to the next phase/sub-phase.

### Pre-Commit Checklist
- [ ] Feature/fix implemented completely
- [ ] Tests added or updated
- [ ] `./gradlew check` passes
- [ ] `./gradlew verifyPlugin` passes
- [ ] Code follows style guidelines
- [ ] No TODO comments left unresolved (for this phase)

## Build & Test Commands

```bash
# Build
./gradlew build                    # Compile + test
./gradlew build -x test            # Fast build (skip tests)
./gradlew clean build              # Full rebuild
./gradlew buildPlugin              # Build plugin ZIP

# Test
./gradlew test                                    # All tests
./gradlew test --tests "ClassName"                # Single class
./gradlew test --tests "ClassName.methodName"     # Single method
./gradlew test --tests "*Pattern*"                # Pattern match

# Verify
./gradlew check                    # All checks (test + verification)
./gradlew verifyPlugin             # Plugin compatibility check

# Run
./gradlew runIde                   # Launch IDE with plugin
```

## Project Structure

```
src/main/
├── kotlin/in/co/spendly/intellibru/  # Source code
│   ├── MyMessageBundle.kt             # i18n bundle
│   └── MyToolWindow.kt                # Compose UI tool window
└── resources/
    ├── messages/MyMessageBundle.properties  # Localization
    └── META-INF/plugin.xml                  # Plugin manifest
src/test/kotlin/                       # Tests (mirror main structure)
```

## Code Style Guidelines

### Package Naming
Use backtick-escaped package names (`in` is a Kotlin keyword):
```kotlin
package `in`.co.spendly.intellibru
```

### Naming Conventions
| Element | Convention | Example |
|---------|------------|---------|
| Classes | PascalCase | `MyToolWindowFactory` |
| Functions | camelCase | `createToolWindowContent()` |
| Composables | PascalCase, private | `private fun MyContent()` |
| Constants | SCREAMING_SNAKE_CASE | `private const val BUNDLE` |
| Variables | camelCase | `val labelText` |

### Import Order
1. `androidx.compose.*` (Compose)
2. `com.intellij.*` (IntelliJ Platform)
3. `org.jetbrains.jewel.*` (Jewel UI)
4. `kotlin.*` / `java.*` (stdlib)

### Visibility
- `internal` for non-public API classes
- `private` for implementation details and Composables
- `@JvmStatic` on companion object methods for Java interop

### Composable UI Pattern
```kotlin
class MyToolWindowFactory : ToolWindowFactory {
    override fun shouldBeAvailable(project: Project) = true

    override fun createToolWindowContent(project: Project, toolWindow: ToolWindow) {
        toolWindow.addComposeTab("Tab Name", focusOnClickInside = true) {
            LaunchedEffect(Unit) { /* initial load */ }
            MyContent()
        }
    }
}

@Composable
private fun MyContent() {
    val state = remember { mutableStateOf(initialValue) }
    Column(Modifier.padding(20.dp), verticalArrangement = Arrangement.spacedBy(8.dp)) {
        Text(state.value)
        OutlinedButton(onClick = { /* handler */ }) { Text("Action") }
    }
}
```

### Message Bundle (i18n)
```kotlin
MyMessageBundle.message("key.name")
MyMessageBundle.lazyMessage("key.name", param1, param2)
```

## Error Handling

```kotlin
import com.intellij.openapi.diagnostic.Logger

private val LOG = Logger.getInstance(MyClass::class.java)

try {
    // risky operation
} catch (e: Exception) {
    LOG.error("Operation failed", e)
}
```
- Use `Logger` for logging (NOT println)
- Use `com.intellij.openapi.ui.Messages` for user dialogs
- Use `ProgressManager` for background tasks with cancellation

## Plugin Configuration

Edit `src/main/resources/META-INF/plugin.xml`:
- **Extensions:** `<extensions defaultExtensionNs="com.intellij">`
- **Actions:** `<actions>`
- **Dependencies:** `<depends>` tags
- **Services:** `<applicationService>` or `<projectService>`

## Testing

```kotlin
import com.intellij.testFramework.fixtures.BasePlatformTestCase

class MyTest : BasePlatformTestCase() {
    fun testSomething() { /* ... */ }
}
```

## Common Pitfalls

1. **Package naming:** Always use backticks around `in` in package declarations
2. **JVM version:** Requires JDK 21 - check JAVA_HOME if build fails
3. **Compose state:** Use `remember { mutableStateOf() }` for UI state
4. **Thread safety:** Use `ApplicationManager.getApplication().invokeLater{}` for UI updates
5. **Plugin reloading:** Code changes require IDE restart via `./gradlew runIde`

## Quick Reference

| Task | Command/Location |
|------|------------------|
| Run all checks | `./gradlew check` |
| Run plugin | `./gradlew runIde` |
| Add extension | `plugin.xml` → `<extensions>` |
| Add i18n string | `MyMessageBundle.properties` |
| Add dependency | `build.gradle.kts` → `intellijPlatform` |
| View logs | `build/idea-sandbox/system/log/idea.log` |

## Resources

- [IntelliJ Platform SDK](https://plugins.jetbrains.com/docs/intellij/)
- [Jewel (Compose for IntelliJ)](https://github.com/JetBrains/jewel)
