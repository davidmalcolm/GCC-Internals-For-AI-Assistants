# GCC Testing Guide for AI Assistants

This document covers testing practices for GCC. See
[GCC-Internals-For-AI-Assistants.md](../GCC-Internals-For-AI-Assistants.md) for general GCC development practices
and AI policy.

**Note:** This is just an experiment by one GCC developer to gain
experience (and share it).  Nothing in here reflects official project policy.

**AI and Testing:** Policy on AI-generated tests is still under
discussion. Currently:

**PERMITTED:**
- Proposing test cases after reviewing a patch
- Suggesting scope of testing ("you should test X, Y, Z")
- Reviewing patches and identifying missing test coverage
- Helping design test strategy

**UNDER DISCUSSION:**
- Whether AI-generated tests can be included in submissions
- Extent of human review required

See main GCC-Internals-For-AI-Assistants.md for overall AI policy. For tests, focus on being
helpful in test design and review rather than just generating code.

## Official Documentation

**Primary resource:** gcc-newbies-guide working-with-the-testsuite.rst
has excellent practical examples of running and debugging tests.

**DejaGnu manual:** Documentation for the test framework

## Test Suite Overview

GCC has two main testing approaches:

**1. DejaGnu (black-box testing):**
- Tests GCC as a whole by invoking the compiler
- Organized by language and feature area
- Best for testing end-to-end compiler behavior

**Test locations:**
- `gcc/testsuite/gcc.dg/` - C tests
- `gcc/testsuite/g++.dg/` - C++ tests
- `gcc/testsuite/gfortran.dg/` - Fortran tests
- `gcc/testsuite/gcc.dg/torture/` - Tests run at multiple optimization
  levels
- `libstdc++-v3/testsuite/` - C++ standard library tests

**2. Unit test framework (white-box testing):**
- Added in last decade (relatively recent for GCC)
- See `gcc/selftest.h`
- Tests GCC's internal data structures directly
- Run via `gcc -fself-test`
- Best for testing internal APIs and data structure invariants

**Choosing between them:**
- **DejaGnu:** Testing compiler behavior, diagnostics, code generation
- **Unit tests:** Testing internal data structures, API contracts,
  helper functions

## Running Tests - Key Patterns

**From build directory:**

```bash
# All tests for a language
make check-gcc      # C compiler tests
make check-g++      # C++ compiler tests

# Specific test pattern (CRITICAL for fast iteration)
make check-gcc RUNTESTFLAGS="dg.exp=pr12345.c"

# Wildcard patterns
make check-gcc RUNTESTFLAGS="dg.exp=analyzer/*.c"

# Parallel execution
make -j8 check-gcc
```

**AI assistant note:** The `RUNTESTFLAGS` pattern is essential for
running individual tests during development. Full test suite can take
hours; individual test takes seconds.

## Test Results - Where to Look

**After running tests:**
```
<build>/gcc/testsuite/gcc/gcc.sum  # Summary
<build>/gcc/testsuite/gcc/gcc.log  # Detailed log
```

**Result codes:**
- `PASS` - Test passed
- `FAIL` - Test failed (investigate)
- `XPASS` - Unexpected pass (test marked as expected to fail, but
  passed)
- `XFAIL` - Expected failure (test is known to fail)
- `UNSUPPORTED` - Test not supported on this target
- `UNRESOLVED` - Test couldn't determine pass/fail
- `ERROR` - Test harness error

**For AI assistants:** When user reports test failure, ask them to
check the `.log` file for detailed output. Don't guess at what went
wrong.

## Writing Tests

**Official reference:** `gcc/doc/sourcebuild.texi` documents the
DejaGnu directives and test framework in detail.

**Quick example:**
```c
/* { dg-do compile } */
/* { dg-options "-fanalyzer" } */

void test(void *ptr)
{
  free(ptr);
  free(ptr); /* { dg-warning "double-'free'" } */
}
```

**For directives, syntax, and details:** See `sourcebuild.texi` and
existing tests in `gcc/testsuite/` for patterns.

## Test Requirements

When suggesting tests to users:

- **Bug fixes:** Include PR number in filename (`pr12345.c`)
- **Keep minimal:** Smallest reproducer possible
- **Portability:** Avoid platform-specific assumptions unless necessary
- **Torture tests:** Tests in `gcc.dg/torture/` run at multiple `-O`
  levels

## Debugging Test Failures

**Useful RUNTESTFLAGS:**
```bash
# Verbose output (shows actual gcc command)
make check-gcc RUNTESTFLAGS="-v -v dg.exp=test.c"

# Keep temporary files for inspection
make check-gcc RUNTESTFLAGS="--keep dg.exp=test.c"
```

See gcc-newbies-guide working-with-the-testsuite.rst for detailed
debugging examples.

## Comparing Test Results

**Tool:** `contrib/compare_tests`

Used to compare test results between control and patched builds.
Identifies new failures and unexpected passes. See gcc-newbies-guide
readying-a-patch.rst for regstrapping workflow.

**MCP tool:* https://builder.sourceware.org/mcp

This online service for LLMs offers quick access to an archive of
analyzed testsuite run results from a variety of projects (including
GCC), and on a variety of platforms.

---

*Last updated: 2026-05-15*
*This is a DRAFT for discussion purposes*
