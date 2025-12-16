---
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(dotnet build:*), Bash(dotnet restore:*), Bash(dotnet test:*), Bash(ng build:*), Bash(ng test:*), Bash(npm run build:*), Bash(npm run test:*), Read, Glob, Grep, Task
description: Review uncommitted changes, build projects, and run unit tests
---

Perform a local code review on uncommitted changes with full build and test verification.

Follow these steps precisely:

1. **Check for uncommitted changes**: Run `git status` to verify there are uncommitted changes. If there are no changes (working tree clean), inform the user and stop.

2. **Detect project type**: Use Glob to find:
   - `.sln` files (search current directory and up to 3 parent levels)
   - `angular.json` files (indicates Angular project)
   - Store the paths for later use

3. **Detect test projects**: Use Glob and Grep to find test projects:
   - Search for `.csproj` files containing `NUnit` or `xunit` or `MSTest` package references
   - Note: Test projects typically have names ending in `.Tests` or `.UnitTests`
   - Store test project paths for later use

4. **Gather CLAUDE.md files**: Use a Haiku agent to find all relevant CLAUDE.md files:
   - Root CLAUDE.md (if exists)
   - Any CLAUDE.md files in directories containing modified files
   - Return a list of file paths (not contents)

5. **Get uncommitted changes**: Run `git diff HEAD` to get all uncommitted changes (both staged and unstaged). Use a Haiku agent to summarize what files changed and the nature of changes.

6. **Launch parallel review agents**: Launch 4 parallel Sonnet agents to independently review the changes. Each agent should return a list of issues with:
   - Issue description
   - File and line number
   - Reason (CLAUDE.md compliance, bug, security, best practice)
   - Confidence score (0-100)

   **Agent #1 - CLAUDE.md Compliance**: Audit changes against CLAUDE.md guidelines. Focus on project-specific rules and conventions.

   **Agent #2 - Bug Detection**: Shallow scan for obvious bugs in the changes only. Focus on:
   - Null reference issues
   - Resource leaks
   - Logic errors
   - Async/await misuse
   - Exception handling problems
   Ignore pre-existing issues and false positives.

   **Agent #3 - .NET Best Practices** (if .sln detected): Review for:
   - Clean Architecture violations
   - SOLID principle violations
   - Async/await anti-patterns
   - Dependency injection issues
   - Entity Framework misuse
   - Security vulnerabilities (SQL injection, XSS, etc.)

   **Agent #4 - Angular Best Practices** (if angular.json detected): Review for:
   - Component design issues
   - RxJS anti-patterns (memory leaks, improper subscriptions)
   - Change detection issues
   - Template security (XSS)
   - Performance issues (large bundles, unnecessary re-renders)

7. **Build .NET project** (if .sln detected):
   - Run `dotnet restore` on the solution
   - Run `dotnet build` on the solution
   - Capture any build errors or warnings
   - If build fails, report the errors with root cause analysis but continue with the review

8. **Build Angular project** (if angular.json detected):
   - Run `ng build` or `npm run build`
   - Capture any build errors
   - If build fails, report the errors with root cause analysis but continue with the review

9. **Run .NET unit tests** (if test projects detected):
   - Run `dotnet test` on detected test projects (or entire solution)
   - Use `--no-build` flag if build already succeeded
   - Capture test results:
     - Total tests
     - Passed tests
     - Failed tests
     - Skipped tests
   - If tests fail, capture failure details (test name, error message, stack trace)

10. **Run Angular tests** (if angular.json detected and test scripts available):
    - Run `ng test --watch=false --browsers=ChromeHeadless` or `npm run test -- --watch=false`
    - Capture test results
    - If tests fail, capture failure details

11. **Score issues**: For each issue found in step 6, launch a parallel Haiku agent to score confidence (0-100):
    - 0: False positive, doesn't stand up to scrutiny, or pre-existing issue
    - 25: Might be real, but could be false positive. Stylistic issue not in CLAUDE.md
    - 50: Real issue but minor or nitpick. Not very important relative to PR
    - 75: Verified real issue that will be hit in practice. Important or explicitly in CLAUDE.md
    - 100: Absolutely certain, confirmed real issue that happens frequently

12. **Filter and report**:
    - Filter out issues with score less than 80
    - Combine with build errors and test results
    - Report to user in the following format:

---

### Local Code Review Results (with Tests)

**Project:** [solution/project name]
**Changes reviewed:** [number of files changed]

#### Build Status
- .NET Build: [PASSED/FAILED - error summary if failed]
- Angular Build: [PASSED/FAILED/NOT APPLICABLE - error summary if failed]

#### Test Results

**.NET Tests:**
- Total: [N] | Passed: [N] | Failed: [N] | Skipped: [N]
- Status: [PASSED/FAILED]

[If failed, list each failed test:]
- `[TestClass].[TestMethod]`: [Error message]

**Angular Tests:**
- Total: [N] | Passed: [N] | Failed: [N]
- Status: [PASSED/FAILED/NOT APPLICABLE]

[If failed, list each failed test:]
- `[Test description]`: [Error message]

#### Code Review Issues

Found [N] issues:

1. **[Brief description]** (Confidence: [score])
   - File: `[file path]:[line number]`
   - Reason: [CLAUDE.md says "..." / Bug / Security / Best Practice]
   - Details: [explanation]

2. ...

#### Summary
- Code review issues: [N]
- Build errors: [N]
- Test failures: [N]
- Overall status: [PASSED/NEEDS ATTENTION]
- Recommendation: [brief recommendation]

---

If everything passed:

---

### Local Code Review Results (with Tests)

**Project:** [solution/project name]
**Changes reviewed:** [number of files changed]

#### Build Status
- .NET Build: PASSED
- Angular Build: PASSED/NOT APPLICABLE

#### Test Results

**.NET Tests:** [N] passed, 0 failed
**Angular Tests:** [N] passed, 0 failed / NOT APPLICABLE

#### Code Review Issues

No significant issues found.

#### Summary
All builds passed. All tests passed. No code review issues found.

---

**Important notes:**
- NEVER apply fixes automatically. Only report issues.
- NEVER modify any files. This is a read-only review.
- Use the `local-code-reviewer` agent skill for .NET/Angular specific knowledge.
- If user wants to fix issues, they must explicitly ask Claude Code to do so.
- Do not check linting or formatting - assume CI will handle those.
- Focus on real bugs and important issues, not nitpicks.
- Test failures are critical - always report them prominently.

**Examples of false positives to filter:**
- Pre-existing issues not in the diff
- Code that looks like a bug but isn't
- Pedantic nitpicks
- Issues linters/compilers will catch
- General code quality issues (unless in CLAUDE.md)
- Issues with lint ignore comments
- Intentional functionality changes
