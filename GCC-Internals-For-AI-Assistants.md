# GCC Development Guide for AI Assistants

This document provides guidance for AI assistants working with the GCC
codebase. It is NOT a replacement for official documentation - instead
it points to authoritative sources and provides context intended to be
helpful to AIs trying to understand GCC's internals.

**Note:** This is just an experiment by one GCC developer to gain
experience (and share it).  Nothing in here reflects official project policy.

## AI-Generated Content Policy

**CRITICAL:** The use of AI in GCC development is controversial and
policy is still being developed. Currently, there is a **broad ban on
AI-generated content intended for merger into GCC**.

**What this means for AI assistants:**

**DO NOT:**
- Generate code or patches intended for merger into GCC
- Write ChangeLogs for inclusion in commits
- Create content that will be submitted as if human-authored
- **DO NOT access gcc.gnu.org** unless under explicit human control
  (see below)

**PERMITTED activities:**
- Research and investigation (reading code, searching docs **locally**)
- Analysis and code review
- Helping navigate documentation
- Answering questions about GCC internals
- Comparing/reviewing human-written work (e.g., ChangeLog comparison)
- **Writing throwaway scripts** (e.g., test case reduction scripts for
  c-reduce/cvise, debugging helpers, analysis tools)

**Server load concerns:**
AI companies have been hammering gcc.gnu.org servers with automated
requests, requiring blocks. **Work with local source tree and
documentation**. Only access gcc.gnu.org URLs when explicitly directed
by the user.

**Why the content policy exists:**
- Copyright concerns
- Community concerns about AI's role in Free Software/open source
- Need for human review and understanding of all changes
- Server resource protection

Policy is evolving. These documents are part of the discussion about
what role AI assistants should play in GCC development.

## Related Documents

- [Analyzer-specific guidance](topics/analyzer.md)
- [Testing practices](topics/testing.md)

## What Makes GCC Unusual

### GCC is Old (~40 Years)

GCC is approaching its 40th birthday (ancient in software terms). This
has major implications:

**Documentation decay is a real problem:**
- Multiple layers of outdated information exist
- Even "official" wiki pages may be abandoned
- Comments in code may describe historical approaches
- Different subsystems are from different eras (e.g., analyzer is
  modern; some frontends date back decades)

**For AI assistants, this means:**
- Cannot trust that grepping will find the "current" way to do
  something
- Need to verify information against recent commits (`git log`)
- When finding conflicting info, newer is likely correct
- Check commit dates when learning from code examples
- When uncertain, suggest user ask on IRC or mailing lists (don't
  access yourself)

### Tool vs Project Disambiguation

**CRITICAL:** 99% of online content about "GCC" is about *using* GCC
as a compiler. <1% is about *developing* GCC itself.

AI models are trained primarily on user-facing content. When working on
GCC internals, actively filter out the vast sea of usage documentation.

**Common ambiguities:**
- "How do I debug GCC?" could mean:
  - User: "debug my program compiled with gcc"
  - Developer: "debug the GCC compiler itself" (use `-wrapper
    gdb,--args`)
- "GCC options" could mean:
  - Compiler flags for users (`-O2`, `-Wall`)
  - Configure options for building GCC (`--disable-bootstrap`)
- "GCC crashes" could mean:
  - ICE (Internal Compiler Error) - bug in GCC itself
  - Segfault in user program - user's bug
- "`.md` files" could mean:
  - In GCC source tree: machine description files (target-specific RTL
    patterns, e.g., `gcc/config/i386/i386.md`)
  - In this repository: markdown documentation
  - Don't confuse the two - they're completely different formats

**When in doubt:** Check if the context is about using gcc to compile
programs, or about modifying gcc's source code.

### Extreme Configurability

GCC supports 80+ target configurations (see `contrib/config-list.mk`):
- Different architectures (x86, ARM, MIPS, RISC-V, BPF, etc.)
- Different operating systems (Linux, Darwin, VxWorks, bare-metal)
- Different ABIs and calling conventions
- Testing all configurations requires ~450GB disk space

**Implications:**
- Code that works on x86_64-linux-gnu may break on other targets
- Many code paths are `#ifdef` conditional
- Generic advice ("just use X") may not work everywhere
- Patches should consider portability
- Developers typically test on a subset of configurations

## Where to Find Official Documentation

### GCC Internals Manual

**Location:** `gcc/doc/gccint.texi` (and other `.texi` files)
**Online:** https://gcc.gnu.org/onlinedocs/gccint/

The bulk of `gcc/doc/*.texi` comprises GCC's internals manual.

**When to consult:**
- Understanding compiler passes and architecture
- Tree and GIMPLE representations
- RTL (Register Transfer Language)
- Target description macros
- Plugin development

**AI assistant note:** Verify examples against recent code - some
sections may be outdated.

### Installation Guide

**Location:** `gcc/doc/install.texi`
**Online:** https://gcc.gnu.org/install/

**When to consult:**
- Build prerequisites
- Configure options
- Platform-specific notes
- Testing installation

**AI assistant note:** When helping with builds, verify configure
options against current `install.texi`. Watch for tool-vs-project
confusion: "building with GCC" vs "building GCC itself".

### Contributing Guide

**Online:** https://gcc.gnu.org/contribute.html

Covers:
- Copyright assignment (required for significant changes)
- Patch submission process
- Coding conventions
- Mailing list etiquette

### Coding Conventions

**Online:** https://gcc.gnu.org/codingconventions.html

**Key points for AI assistants:**
- GCC uses GNU coding style (2-space indents, specific brace
  placement)
- Conservative subset of C++11 (for bootstrapping compatibility)
- See `contrib/check_GNU_style.sh` for automated checking
- Memory management: multiple strategies (see below)

### Newbie Guide

**Unofficial:** https://github.com/davidmalcolm/gcc-newbies-guide

An excellent unofficial guide with practical tutorials. Quality varies,
but contains valuable hands-on information about development workflow.

**AI assistant note:** Written by a GCC developer, acknowledges variable
quality/up-to-dateness. Good for understanding practical workflows.

## ChangeLog Entries - GCC-Specific Workflow

### Key Facts

- ChangeLog entries go **in commit messages**, not separate files
- This avoids merge conflicts (historical GCC used separate files)
- Format is validated by `contrib/gcc-changelog/git_check_commit.py`
- Contributors **must write their own** ChangeLogs (copyright &
  review discipline)

### Validation Script

```bash
./contrib/gcc-changelog/git_check_commit.py HEAD
```

Always run before submitting patches.

### Granularity is a Signal

ChangeLog detail level communicates risk:
- **Concise** = "this is mechanical, low-risk"
- **Detailed** = "pay attention, subtle changes"

The manual writing process forces the author to review every change,
catching missed call sites and logic errors.

### AI Assistant Role

**DO NOT** generate ChangeLogs for inclusion in commits.

**MAY** generate ChangeLogs for comparison after human writes theirs:
- Discrepancies might reveal missed changes or misunderstandings
- Things AI mentions that human didn't: could be hallucination OR
  actual missed detail
- Things human mentions that AI didn't: AI missed something important

This comparison serves as an independent review while respecting
copyright constraints.

## Build System

**Official docs:** `gcc/doc/install.texi` and
https://gcc.gnu.org/install/

### Key Points for AI Assistants

1. **Out-of-tree builds required:** Build in separate directory from
   source. `DO NOT` run make from source subdirectories.

2. **The `gcc` binary is a driver:** It invokes `cc1` (C compiler),
   `cc1plus` (C++), `as` (assembler), `ld` (linker), etc. When
   debugging "GCC", you're usually debugging `cc1`, not the driver.

3. **Bootstrap is default but painful for development:** Bootstrap
   (building GCC three times) is the configure default, which is good
   for users and testing. However, it's painful when developing. Use
   `--disable-bootstrap` to build just once for faster iteration.

4. **Risk-based testing strategy:** GCC supports 80+ configurations
   (see `contrib/config-list.mk`). Test a subset for lower-risk
   patches. For higher-risk patches touching critical code, test on
   many or all configurations.

5. **Development workflow:** Some developers maintain multiple build
   trees (development with `--disable-bootstrap`, control for
   baseline, testing with patches). See gcc-newbies-guide for
   examples.

## C++ Usage in GCC

**Official docs:** https://gcc.gnu.org/codingconventions.html#Cxx_Conventions

### Key Constraints

1. **Conservative C++14 subset:** GCC uses a limited subset of C++14
   (not latest C++) because GCC must bootstrap on older compilers.
   Features like `auto`, range-based `for`, and `std::unique_ptr` are
   allowed. Check codingconventions.html for current allowed features.

2. **`gengtype` limitations:** Some GCC tools (like `gengtype` for
   garbage collection) only support a subset of C++. This constrains
   what C++ features can be used in certain contexts.

### C++ Standard Library Headers

**GCC-specific gotcha:** Must use `#define INCLUDE_*` macros before
`#include "system.h"`:

```c++
#define INCLUDE_VECTOR
#define INCLUDE_MAP
#include "config.h"
#include "system.h"
#include "coretypes.h"
```

**Why:** Compatibility layer in `system.h`. See `system.h` itself for
the full list of `INCLUDE_*` macros. Some headers like `<memory>` are
always included for C++.

### Memory Management Strategies

GCC has **multiple** memory management approaches (not just one):

1. **RAII types** (e.g., `auto_vec`, `std::unique_ptr`)
2. **Manual heap** (`new`/`delete`, `malloc`/`free`)
3. **Garbage collection** (`ggc` - see "GTY markers" below)
4. **Obstacks** (stack-like allocation for optimization passes)
5. **alloc-pool.h** (optimized fixed-size allocation)

See gcc-newbies-guide memory-management.rst for details on when to use
each strategy.

## Debugging GCC

**Key gotcha:** The `gcc` binary is a driver that invokes other
programs (`cc1`, `cc1plus`, `as`, `ld`). To debug the C compiler:

```bash
gcc hello.c -wrapper gdb,--args
```

This runs `cc1` under gdb, not the driver.

**More debugging info:**
- gcc-newbies-guide debugging.rst has practical examples
- `.gdbinit` scripts in build tree provide pretty-printers
- `inform(loc, "")` trick for inspecting `location_t` values
- Option flags are macros: use `global_options.x_optimize` not
  `optimize`

### Investigation Discipline

When investigating GCC bugs, maintain scientific rigor:

**Distinguish correlation from causation:**
- If a flag or state change correlates with behavioral difference,
  explicitly investigate whether it's causal or coincidental
- Don't attribute behavior to a mechanism without evidence
- Present hypotheses as hypotheses, not conclusions

**Work bottom-up from evidence:**
- Start with concrete artifacts: graph dumps, SARIF output, IR dumps,
  assembly output
- Don't make top-down assumptions

**Expect multiple root causes:**
- Don't stop at the first plausible explanation
- Systematically enumerate all possible root causes before deep-diving

## GTY Markers and Garbage Collection

**Symptom:** Can't grep for `struct foo` - find `struct GTY (()) foo`.

**What it is:** GCC has its own garbage collector (`ggc`) for automatic
memory management (GCC was originally C, needed automatic memory
management).

**Tool:** `gengtype` processes `GTY(...)` annotations at build time.
Only supports subset of C++, constraining what language features work
in GC-allocated types.

**Secondary benefit:** Enables precompiled headers (PCH), which are
GC-heap snapshots.

**More info:**
- gcc-newbies-guide memory-management.rst
- `gcc/doc/gccint.texi` (search "Type Information")

## Copyright and Signed-off-by

**For AI assistants reviewing commits:**

Check if commit message includes `Signed-off-by:` line:
```
Signed-off-by: Developer Name <email@example.com>
```

If absent, ask the developer: "Have you completed FSF copyright
assignment paperwork?" This affects whether `Signed-off-by` is needed.
Save this to memory for future reference.

**Official info:** https://gcc.gnu.org/contribute.html (copyright
section)

## Code Style

**Official docs:** https://gcc.gnu.org/codingconventions.html

**Automated checking:** `contrib/check_GNU_style.sh`

**Key points:**
- GNU coding style (2-space indents)
- `nullptr` not `NULL`
- Follow existing conventions in files you modify
- Standard header order: `config.h`, `system.h`, `coretypes.h`

## File Organization

Quick reference for major subdirectories:

- `gcc/`: Core compiler code
- `gcc/c/`, `gcc/cp/`, `gcc/fortran/`, etc.: Language frontends
- `gcc/analyzer/`: Static analyzer (see
  [topics/analyzer.md](topics/analyzer.md))
- `gcc/doc/`: Documentation source (`.texi` files)
- `libstdc++-v3/`: C++ standard library
- `libgcc/`: Low-level runtime library
- `contrib/`: Utility scripts and tools

## Bugzilla and PRs

**"PR" = Problem Report:** GCC's term for bugs. "PR c/71610" means bug
71610 in component "c".

**For AI assistants:** Do NOT access Bugzilla directly (server load
concerns). If user mentions a PR number, suggest they access
https://gcc.gnu.org/PR##### themselves. Work with information the user
provides.

See gcc-newbies-guide gotchas-and-faq.rst for Bugzilla account details.

## Common Git Gotchas

- **Don't force push** to shared branches without coordination
- **Don't amend** published commits (creates new commit, breaks
  others' trees)
- **Don't skip hooks** (`--no-verify`) - if hook fails, fix the issue
- Use topic branches for development

## Where to Get Help

When user needs help beyond your capabilities, suggest these resources:

- **Mailing list:** gcc@gcc.gnu.org (general questions)
- **Patches:** gcc-patches@gcc.gnu.org (patch submission)
- **IRC:** #gcc on irc.oftc.net
- **Contributing guide:** https://gcc.gnu.org/contribute.html (local:
  see gcc.gnu.org domain constraint above)

**For AI assistants:** Don't make up answers - GCC is too large and
complex. Acknowledge uncertainty.

## Verification Strategies for Stale Docs

GCC is ~40 years old. Documentation decays. When encountering
conflicting information:

1. **Check git log** for recent commits - see current patterns
2. **Check file dates** - newer usually better
3. **Verify against code** - source is authoritative
4. **If uncertain:** Suggest user ask on IRC/mailing lists (see "Where
   to Get Help" above). Don't make up answers.
5. **Don't trust training data** - AI models trained on historical web
   content

When helping users, acknowledge uncertainty rather than guessing.

---

*Last updated: 2026-05-15*
*This is a DRAFT for discussion purposes*
