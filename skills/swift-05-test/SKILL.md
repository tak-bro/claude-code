---
name: swift-05-test
description: "iOS/Swift test writing and execution. Triggers on 'swift test', '스위프트 테스트', 'ios test', 'iOS 테스트', 'swiftui test', 'xctest'."
model: sonnet
tools: Read, Bash, Glob, Grep, Write, Edit
---

# Swift Test

Write and run tests for iOS/Swift features.

**Agents:** test-strategy-expert, swift-ios-expert

## Pre-Test

1. Implementation complete? (`/swift-02-implement`)
2. Review passed? (`/swift-03-review` -> `/swift-04-fix`)
3. Files to test identified

---

## Test Strategy

1. **Analyze implemented files** — prioritize by criticality:
   - [Critical] ViewModel, Repository, UseCase, Mapper
   - [Important] View (Preview + UI test)
   - [Nice] Component (optional)

2. **Mocking targets**: Repository (protocol mock), API Service (URLProtocol/protocol mock), Local storage (in-memory), UserDefaults (ephemeral suite)

---

## Rules

- **Do**: AAA pattern, naming `test_[action]_[condition]_[expected]`, async/await, protocol-based mocks, `@MainActor` for ViewModel tests, test error/edge cases
- **Don't**: snapshot tests, implementation detail tests, share state, `sleep()`/`Task.sleep()`, test private methods

**Maintain 100% compatibility with existing test stack and patterns.**

---

## Run and Verify

```bash
xcodebuild test -workspace {Name}.xcworkspace -scheme {Scheme} -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16'
swift test --verbose
```

-> All tests pass -> Complete
-> Failures -> `/swift-04-fix`
