# Android Code Review - Agent Guide

## Quick Start

Use this skill when:
- Reviewing pull requests on Android projects
- Analyzing code for quality issues
- Validating architecture pattern adherence
- Checking for security vulnerabilities
- Assessing performance implications

## Common Tasks

### 1. Review a Pull Request

**Input:** "Review this PR with focus on architecture"

**Action:** Analyze code systematically:

```kotlin
// Checklist:
□ Architecture pattern adherence (MVVM/MVI/MVP)
□ State management correctness
□ Coroutine scope usage
□ Compose recomposition efficiency
□ Security considerations
□ Test coverage
```

### 2. Security Review

**Input:** "Check this code for security issues"

**Action:** Look for:
- Hardcoded credentials
- Insecure data storage
- Network security misconfigurations
- Input validation gaps
- Proguard/R8 configuration issues

### 3. Performance Review

**Input:** "Review for performance issues"

**Action:** Check:
- Unnecessary recompositions in Compose
- Memory leak risks
- Inefficient collection operations
- Coroutine dispatcher misuse
- Database query efficiency

## Severity Levels

| Level | Action |
|-------|--------|
| Critical | Block merge - security/crash/data loss |
| Important | Request changes - architecture/performance |
| Suggestion | Comment - style/minor optimization |

## Review Template

```markdown
## Summary
- [ ] Critical issues found
- [ ] Important issues found
- [ ] Suggestions provided

## Issues Found

### [Critical] Issue Title
Description and impact.

```kotlin
// Current code
// Problem explanation
```

```kotlin
// Recommended fix
```

## Positive Findings
- Good pattern usage
- Clean implementation areas
```
