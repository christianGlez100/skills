---
name: kmp-code-quality
description: Perform a comprehensive SonarQube-style static code quality audit of Kotlin Multiplatform projects. Analyze every relevant source file, identify bugs, security vulnerabilities, code smells, technical debt, best-practice violations, and missing or inadequate tests. Report stable rule IDs, severity, type, exact verified file paths and line numbers, evidence, remediation guidance, effort estimates, and test status. Generate or overwrite docs/code-quality/CODE_QUALITY_REPORT.md from the current code only. Never compare with previous reports.
---

# KMP Code Quality — SonarQube-style Audit

## 1. Mission

Perform a complete, evidence-based static code quality audit of the current Kotlin Multiplatform project, using a report structure inspired by SonarQube.

This is a custom audit, not the official SonarQube engine. Do not claim that SonarQube ran, that its Quality Gate passed, or that official Sonar metrics were measured unless an actual SonarQube analysis provides that evidence.

The audit must identify:
- Bugs and correctness risks.
- Security vulnerabilities and security hotspots requiring review.
- Code smells and maintainability issues.
- Architecture and SOLID problems.
- Kotlin, Compose, Coroutines, Flow, and KMP issues.
- Missing or inadequate unit tests.
- Technical debt and actionable improvements.
- Positive practices supported by source evidence.

Every actionable issue must provide its rule ID, severity, type, exact file location, affected class/function, evidence, explanation, remediation recommendation, and expected benefit.

Analyze the current source code from scratch on every execution. Do not use previous reports as a baseline, retain historical findings, or compare runs.

Do not modify production code unless the user explicitly requests implementation of fixes.

## 2. Required workflow

1. Identify the project root.
2. Inspect repository structure, modules, Gradle configuration, version catalogs, dependencies, targets, and source sets.
3. Identify the architecture actually implemented.
4. Enumerate all relevant production files and test files.
5. Map production files and declarations to related tests.
6. Inspect every relevant production source file individually.
7. Review tests and assess whether their coverage is adequate.
8. Review project-wide dependencies, architecture, security, performance, and maintainability.
9. Verify every finding against the current source before reporting it.
10. Record exact current line numbers using actual file contents or line-numbered output.
11. Assign rule ID, type, severity, confidence, remediation, and effort estimate.
12. Recompute all issue counts and report metrics from the current audit.
13. Generate the complete report independently.
14. Create docs/code-quality/ if missing.
15. Overwrite docs/code-quality/CODE_QUALITY_REPORT.md.
16. Verify the report exists at the exact path.
17. Check report consistency, completeness, and absence of stale findings.
18. Summarize results and limitations.

## 3. SonarQube-style classification

Use these issue types:

- BUG: implementation that demonstrably produces or can produce incorrect behavior.
- VULNERABILITY: a supported security weakness with a plausible exploitable consequence.
- SECURITY_HOTSPOT: security-sensitive code requiring contextual review; not automatically a confirmed vulnerability.
- CODE_SMELL: a maintainability, readability, complexity, duplication, or design problem.
- TEST_GAP: missing or inadequate tests for meaningful behavior.
- ARCHITECTURE: a significant architectural or dependency-boundary problem.
- TECHNICAL_DEBT: an identified maintenance burden that needs remediation.
- IMPROVEMENT: a worthwhile but non-mandatory improvement.
- POSITIVE: an evidence-based good practice; do not count this as an issue.

Use one primary type per finding. A security defect can be classified as VULNERABILITY even if it also affects maintainability.

### Severity mapping

Use these custom severity levels inspired by SonarQube:

- BLOCKER: severe issue likely to prevent a critical workflow or cause a major failure; use rarely and only with strong evidence.
- CRITICAL: serious correctness, security, or reliability risk with substantial potential impact.
- MAJOR: significant defect, architectural issue, or maintainability problem requiring planned correction.
- MINOR: localized issue with limited impact or a lower-risk maintainability improvement.
- INFO: informational observation or optional improvement.

Do not equate these custom severities with official SonarQube ratings.

Severity must reflect actual impact and likelihood, not style preference. A missing test is not automatically BLOCKER or CRITICAL. Explain the reason for each high-severity classification.

Use confidence HIGH, MEDIUM, or LOW separately from severity.

## 4. Stable rule IDs

Assign a stable, descriptive rule ID to each distinct rule. Prefer the following prefixes:

- KMP-BUG-xxx: correctness and runtime defects.
- KMP-SEC-xxx: security issues and hotspots.
- KMP-ARCH-xxx: architecture and dependency direction.
- KMP-SOLID-xxx: SOLID and separation of responsibilities.
- KMP-KOTLIN-xxx: Kotlin language and idioms.
- KMP-COMPOSE-xxx: Compose Multiplatform.
- KMP-CORO-xxx: coroutines and concurrency.
- KMP-FLOW-xxx: Flow and reactive state.
- KMP-PLATFORM-xxx: KMP source sets and platform isolation.
- KMP-DI-xxx: dependency injection.
- KMP-NET-xxx: networking and serialization.
- KMP-DATA-xxx: persistence, mapping, and caching.
- KMP-PERF-xxx: performance.
- KMP-TEST-xxx: missing or inadequate tests.
- KMP-GRADLE-xxx: build configuration and dependency management.
- KMP-MAINT-xxx: maintainability, duplication, complexity, and technical debt.
- KMP-DOC-xxx: meaningful documentation issues.

Use an existing rule ID for the same underlying rule whenever appropriate. Do not invent multiple IDs for the same issue merely because it appears in several files. If a rule affects multiple locations, report distinct locations under the same rule where appropriate.

Rule IDs are identifiers for this custom Skill, not official Sonar rule IDs. Never claim equivalence with an official Sonar rule unless independently verified.

## 5. Mandatory file-level analysis

For every relevant production file, record:
- Relative path, module, source set, package, and main class/type/function.
- Issues and improvement opportunities.
- Related test files.
- Test status.
- Exact verified issue locations.
- Rule IDs and severities.
- Remediation recommendations.

Inspect applicable *.kt, *.kts, build configuration, and other relevant source files.

Include classes, interfaces, data classes, sealed types, enums, type aliases, extension functions, top-level declarations, composables, ViewModels, repositories, data sources, DTOs, entities, mappers, use cases, services, DI modules, navigation, utilities, expect/actual declarations, and platform implementations.

Every relevant production file must appear in the file inventory, even if it has no significant issues.

Generated and third-party code may be excluded from normal findings when appropriate. Document exclusions and reasons.

## 6. Current-source verification

This section is mandatory.

For every potential issue:

1. Inspect the current source file.
2. Locate the affected class, function, or declaration.
3. Confirm that the behavior or design problem still exists.
4. Verify the current line number or line range.
5. Confirm the recommendation addresses the actual issue.
6. Include the finding only when supported by current evidence.

Never copy findings, counts, test statuses, or line references from the previous report without independent verification.

If a previously reported issue has been resolved, exclude it from active findings. Do not preserve it as history and do not add a FIXED status.

If a correction is partial, report only the remaining problem at its current location.

If evidence is insufficient, state the uncertainty and do not present the issue as a confirmed defect.

Never guess line numbers. If exact lines cannot be verified, state that clearly and provide the file and declaration location.

Generate a clean report from the current audit results. Do not append to the old report.

## 7. Engineering rules to inspect

### Architecture and SOLID

Inspect separation of concerns, dependency direction, cohesion, coupling, testability, SRP, OCP, LSP, ISP, DIP, circular dependencies, unnecessary abstractions, business logic in UI layers, infrastructure leakage into domain logic, and DTO leakage.

Identify the architecture actually present. Do not assume Clean Architecture. Explain the actual problem and propose a proportionate solution.

### Kotlin and Clean Code

Inspect naming, readability, complexity, responsibilities, duplication, visibility, null safety, immutability, error handling, API clarity, idiomatic Kotlin, collection operations, scope functions, extensions, sealed types, data classes, and mutable-state exposure.

Do not mechanically flag !!, var, long functions, comments, when expressions, or missing abstractions. Require evidence of a meaningful issue.

### Coroutines and Flow

Inspect structured concurrency, scope ownership, cancellation, dispatcher selection, exception handling, blocking calls, resource leaks, lifecycle safety, GlobalScope misuse, StateFlow, SharedFlow, stateIn/shareIn, state exposure, flow transformations, sharing policies, and duplicate work.

### Compose Multiplatform

Inspect state ownership, state hoisting, remember/rememberSaveable, effects and effect keys, recomposition, stability, expensive calculations, UI/business separation, navigation, lifecycle integration, resources, and maintainability.

Do not claim performance defects without evidence.

### Kotlin Multiplatform

Inspect the source sets that actually exist, including commonMain, commonTest, androidMain, androidUnitTest, androidHostTest, iosMain, iosTest, jvmMain, jvmTest, jsMain, jsTest, wasmJsMain, wasmJsTest, webTest, and project-specific source sets.

Check platform API leakage, dependency placement, duplication, expect/actual correctness, target compatibility, shared logic, and platform-specific behavior.

### DI, networking, and data

Inspect Koin or other DI frameworks for scopes, bindings, lifetimes, cycles, and testability.

Inspect Ktor or other networking code for timeouts, authentication, retries, serialization, error handling, HTTP status handling, cancellation, and logging.

Inspect databases, DataStore, Firebase, Supabase, and caches for mapping, data integrity, error handling, offline behavior, synchronization, and sensitive data.

### Security

Search applicable source and configuration files for hardcoded credentials, tokens, API keys, passwords, sensitive logging, insecure storage, and unsafe configuration.

Never include actual secret values in the report. If exposure is suspected, describe the location without disclosing the value and recommend secure configuration and rotation where appropriate.

Distinguish confirmed vulnerabilities from security hotspots requiring manual contextual review.

### Performance

Investigate evidence of blocking calls, repeated expensive computations, redundant requests, unnecessary allocations, inefficient database operations, and avoidable recompositions.

Distinguish measured problems from potential risks. Do not claim measured improvements or regressions without measurements.

### Gradle, documentation, and technical debt

Review plugin and dependency configuration, source-set dependencies, module boundaries, build conventions, deprecated APIs, meaningful TODO/FIXME markers, workarounds, duplication, and documentation of complex or public APIs.

Do not classify every TODO or missing comment as a problem without context.

## 8. Unit test assessment

Every relevant production file must have one status:

- YES: relevant tests exist and adequately cover important behavior.
- PARTIAL: tests exist but meaningful scenarios remain uncovered.
- NO: testable logic exists but no relevant tests were found.
- NOT APPLICABLE: the file genuinely has no meaningful unit-testable behavior.
- UNKNOWN: evidence is insufficient to determine the relationship.

Search applicable test source sets and map tests by class, function, package, feature, naming, and tested behavior.

Evaluate success paths, error paths, edge cases, nullability, business rules, state transitions, mappings, repositories, coroutine cancellation, Flow emissions, and platform-specific behavior as relevant.

Test existence alone is not proof of adequacy. Do not mark complex logic NOT APPLICABLE merely because tests are missing.

For each test gap, provide the production file, class/function, test status, related test files, missing scenarios, and concrete recommendations.

Prefer the existing testing framework. Consider commonTest for shared logic and platform-specific source sets where needed.

Do not claim a measured code coverage percentage without execution-based coverage data.

## 9. Finding format

Every significant issue must follow this template:

### [KMP-RULE-ID] [SEVERITY] [TYPE] Concise title

- **File:** relative/path/File.kt
- **Lines:** verified current line or line range.
- **Module/source set:** actual module and source set.
- **Class/function:** affected declaration.
- **Rule:** custom rule ID and rule rationale.
- **Type:** BUG, VULNERABILITY, SECURITY_HOTSPOT, CODE_SMELL, TEST_GAP, ARCHITECTURE, TECHNICAL_DEBT, or IMPROVEMENT.
- **Severity:** BLOCKER, CRITICAL, MAJOR, MINOR, or INFO.
- **Confidence:** HIGH, MEDIUM, or LOW.
- **Current situation:** actual implementation.
- **Evidence:** specific source behavior supporting the finding.
- **Impact:** practical consequence or risk.
- **Recommended fix:** concrete implementation steps.
- **Example:** concise before/after code when useful.
- **Effort estimate:** S, M, L, or UNKNOWN.
- **Expected benefit:** expected improvement.
- **Verification:** how to confirm the correction.

Effort estimates are qualitative planning estimates, not measured repair time:
- S: localized, straightforward change.
- M: moderate refactoring or coordinated changes.
- L: broad refactoring or multiple modules.
- UNKNOWN: insufficient information.

Do not assign artificial monetary debt costs or remediation minutes unless there is a defensible project-specific basis.

## 10. SonarQube-style metrics

Calculate metrics only from the current audit and explain their scope.

### Issue counts
- Total active findings.
- Findings by severity.
- Findings by type.
- Findings by category.
- Findings by module.
- Findings by source set.
- Findings by rule ID.

### File and test metrics
- Production files discovered and analyzed.
- Files excluded and requiring follow-up.
- Files with YES, PARTIAL, NO, NOT APPLICABLE, and UNKNOWN test status.
- Test gaps by severity.
- Classes with no tests or inadequate tests.

### Reliability, security, and maintainability summaries

Provide separate qualitative summaries for:
- Reliability: correctness defects and supported runtime risks.
- Security: confirmed vulnerabilities and separately identified hotspots.
- Maintainability: code smells, architecture issues, and technical debt.

Do not calculate official SonarQube ratings, Quality Gate status, security ratings, or reliability ratings from custom heuristic findings. Do not claim official cyclomatic complexity, duplication, coverage, or debt metrics unless the corresponding measurements were actually collected.

If a metric cannot be measured, report NOT MEASURED rather than estimating it without evidence.

Do not assign an overall project quality score, letter grade, or unsupported percentage.

## 11. Required report structure

Generate docs/code-quality/CODE_QUALITY_REPORT.md with these sections:

1. Project Information.
2. Executive Summary.
3. Quality Overview.
4. Issues by Severity.
5. Issues by Type.
6. Issues by Module and Source Set.
7. Rule Catalog and Findings.
8. File Inventory.
9. File-Level Audit.
10. Detailed Issues with Evidence and Remediation.
11. Unit Test Coverage Matrix.
12. Test Gap Analysis.
13. Security Review.
14. Architecture and SOLID Review.
15. Best Practices Review.
16. Positive Findings.
17. Technical Debt and Improvement Opportunities.
18. Priority Remediation Plan.
19. Audit Completeness.
20. Excluded Files.
21. Analysis Limitations.

### Quality overview example

Use an illustrative table format, filling values only with actual findings:

| Metric | Result | Scope |
|---|---:|---|
| Bugs | Actual count | Current audit |
| Vulnerabilities | Actual count | Confirmed only |
| Security hotspots | Actual count | Requires contextual review |
| Code smells | Actual count | Current audit |
| Test gaps | Actual count | Missing/inadequate tests |
| Production files analyzed | Actual count | Relevant production files |
| Files requiring follow-up | Actual count | Incomplete verification |

### Issues table

| Rule ID | Severity | Type | File | Lines | Class/Function | Effort |
|---|---|---|---|---|---|---|

### File-level audit table

| File | Source Set | Class/Type | Issues | Improvements | Test Status | Related Test |
|---|---|---|---:|---:|---|---|

### Unit test matrix

| Production File | Class | Test Status | Related Test | Test Source Set | Missing Coverage | Recommendation |
|---|---|---|---|---|---|---|

### Rule catalog

For every custom rule ID used, document:
- Rule ID.
- Title and intent.
- When it applies.
- Why it matters.
- Recommended correction.

## 12. Prioritization

Order recommendations by practical risk:
1. Confirmed critical security and correctness problems.
2. High-impact reliability and data-integrity problems.
3. Significant architecture and dependency issues.
4. Important missing tests for high-risk logic.
5. Maintainability and complexity issues.
6. Lower-risk improvements and optional refinements.

Explain prioritization using evidence and context. Do not imply that a missing test is itself a confirmed runtime defect.

Separate required fixes from recommended and optional improvements.

## 13. Completeness and limitations

- Do not silently sample files during a full audit.
- Track discovered, analyzed, excluded, and unverified files separately.
- If analysis is incomplete, identify the files or source sets not analyzed.
- Document generated and third-party exclusions.
- Do not invent source lines, findings, tests, dependencies, or execution results.
- If build or test execution is available, report actual commands and results.
- If builds/tests were not run, say so explicitly.
- Never disclose secret values.
- Do not modify production source during a normal audit.
- Do not compare with previous reports.
- Do not include resolved historical issues.

## 14. Clean report generation

The canonical report path is:

docs/code-quality/CODE_QUALITY_REPORT.md

Requirements:
1. Create docs/code-quality/ if it does not exist.
2. Build the report from the current audit results.
3. Overwrite CODE_QUALITY_REPORT.md on every execution.
4. Never append to the previous report.
5. Never use .artifacts, .gemini, .android-studio, build, build/reports, reports, or tmp as the canonical final location.
6. Do not create dated copies or alternative report names.
7. Verify that the exact file exists after writing.
8. Check that counts, severity tables, file inventory, and test matrix are internally consistent.
9. Check that every active finding has current evidence and verified line references where possible.
10. Check that resolved issues have not been copied from the previous report.
11. If writing fails, report the failure honestly.

The previous report may be overwritten but must not be read as a historical baseline.

## 15. Final response

Respond in the user's language.

Summarize:
- Production files discovered, analyzed, excluded, and pending.
- Current findings by severity and type.
- Test status counts and important test gaps.
- Major remediation priorities.
- Whether builds/tests were actually executed.
- Important limitations.
- Exact report path: docs/code-quality/CODE_QUALITY_REPORT.md.

This Skill provides a SonarQube-inspired report, not an official SonarQube scan. Clearly distinguish custom static-review findings from metrics produced by actual analysis tools.
