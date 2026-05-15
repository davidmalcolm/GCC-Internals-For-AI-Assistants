# GCC Static Analyzer - AI Assistant Guide

This document provides analyzer-specific guidance to AI assistants. See
[GCC-Internals-For-AI-Assistants.md](../GCC-Internals-For-AI-Assistants.md) for general GCC development practices
and AI policy.

**Note:** This is just an experiment by one GCC developer to gain
experience (and share it).  Nothing in here reflects official project policy.

**REMINDER:** Broad ban on AI-generated code for merger. See main
GCC-Internals-For-AI-Assistants.md for policy details.

## Overview

The GCC static analyzer (`-fanalyzer`) performs path-sensitive analysis
to detect bugs at compile time.

**Location:** `gcc/analyzer/`

**Key characteristic:** The analyzer is relatively modern (compared to
GCC's 40-year history), so documentation is generally more current than
for older subsystems.

**Investigation discipline:** See "Investigation Discipline" section in
[GCC-Internals-For-AI-Assistants.md](../GCC-Internals-For-AI-Assistants.md) for general debugging principles.

## Testing

**Test locations:**
- `gcc/testsuite/gcc.dg/analyzer/` - C tests
- `gcc/testsuite/g++.dg/analyzer/` - C++ tests

**Running tests:**
```bash
# From build directory
make check-gcc RUNTESTFLAGS="analyzer.exp"
make check-g++ RUNTESTFLAGS="analyzer.exp"

# Specific test file
make check-gcc RUNTESTFLAGS="analyzer.exp=malloc-1.c"
```

**More on testing:** See [testing.md](testing.md)
for DejaGnu details.

## Code Organization

**See:** `gcc/doc/analyzer.texi` for documentation of the analyzer's
architecture, code organization, and for hints on debugging it.

**Location:** All analyzer code lives in `gcc/analyzer/`

## Performance Issues

- Analyzer can be slow on large functions with complex control flow
- See `gcc --help=analyzer` for options to limit exploration

---

*Last updated: 2026-05-15*
*This is a DRAFT for discussion purposes*
