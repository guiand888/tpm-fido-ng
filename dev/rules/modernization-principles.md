# Modernization Principles

## Core Philosophy

tpm-fido modernization prioritizes **minimal, necessary changes** over comprehensive rewrites. We update what needs updating for security and compatibility, leaving working code untouched.

## Guidelines

### What We Change

- **Dependency versions**: Update to latest stable versions for security patches
- **Build toolchain**: Upgrade Go version when required for compatibility
- **CI/CD pipelines**: Modernize testing and deployment workflows
- **Security concerns**: Address critical issues (e.g., embedded private keys)
- **Compatibility**: Fix issues that prevent use with modern browsers/systems

### What We Don't Change

- **Working code**: If it compiles, tests pass, and works correctly, leave it alone
- **Architecture**: No refactoring for "improvements" or style preferences
- **Marginal optimizations**: Skip performance tweaks that don't solve real problems
- **Code style**: Don't reformat or reorganize for consistency unless fixing bugs
- **Feature additions**: Only add features that solve documented problems

### Decision Framework

Before making any change, ask:

1. **Is this necessary?** Does it fix a bug, improve security, or enable compatibility?
2. **Is this minimal?** Can we achieve the goal with the smallest possible change?
3. **Will this break anything?** Do we have tests to verify compatibility?
4. **Is this documented?** Will future maintainers understand why this change exists?

### Acceptable Changes

- Updating `go.mod` to newer dependency versions
- Updating Go version in `go.mod` and CI/CD
- Fixing security vulnerabilities (e.g., externalizing private keys)
- Adding missing tests for existing functionality
- Updating documentation to reflect current state
- Fixing bugs that prevent compilation or functionality

### Unacceptable Changes

- Rewriting functions for "cleaner" code
- Reorganizing package structure
- Refactoring for design patterns
- Adding new features not requested
- Changing code style or formatting
- Removing "dead code" without verification

## Implementation Pattern

1. **Identify the problem**: What specifically needs modernizing?
2. **Minimal fix**: What's the smallest change that solves it?
3. **Verify**: Do tests pass? Does it compile? Does it work?
4. **Document**: Update memory bank and commit message explaining why
5. **Move on**: Don't optimize further unless there's a specific problem

## Example: Dependency Update

**Problem**: `golang.org/x/crypto` is from May 2021; needs security patches

**Minimal fix**: Run `go get -u golang.org/x/crypto` and verify tests pass

**What NOT to do**: Refactor code to use new crypto APIs, reorganize imports, or update unrelated dependencies

## Security Exception

Security issues are the only exception to "minimal changes":
- Embedded private keys must be externalized (even if it requires more changes)
- Vulnerability fixes take priority over minimal-change principle
- But still: fix the vulnerability with minimal code changes, not a rewrite
