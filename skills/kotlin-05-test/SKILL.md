---
name: kotlin-05-test
description: "Android/Kotlin test writing and execution. Triggers on 'kotlin test', '코틀린 테스트', 'android test', '안드로이드 테스트', 'kt test', 'android unit test'."
model: sonnet
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Kotlin Test

Write and run tests for Android/Kotlin features.

**Agents:** test-strategy-expert, kotlin-android-expert

## Pre-Test

1. Implementation complete? (`/kotlin-02-implement`)
2. Review passed? (`/kotlin-03-review` -> `/kotlin-04-fix`)
3. Files to test identified

---

## Test Strategy

1. **Analyze implemented files** — prioritize by criticality:
   - [Critical] ViewModel, Repository, UseCase, Mapper
   - [Important] Screen (Composable UI test)
   - [Nice] Component (optional)

2. **Mocking targets**: Repository (MockK), API Service (MockK/MockWebServer), Room DAO (in-memory), DataStore (test), Dispatchers (TestDispatcher)

---

## Rules

- **Do**: AAA pattern, backtick naming (`` `should [action] when [condition]` ``), `runTest` for coroutines, Turbine for Flow, TestDispatcher, test error/edge cases
- **Don't**: snapshot tests, implementation detail tests, share state, `delay()`/`Thread.sleep()`, test Android framework in unit tests

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Run and Verify

```bash
./gradlew test{Variant}UnitTest --tests "*FeatureName*"
./gradlew jacocoTestReport
```

-> All tests pass -> Complete
-> Failures -> `/kotlin-04-fix`
