# GCC Internals for AI Assistants

This repository provides guidance for AI assistants working with the GCC
codebase. It captures GCC-specific gotchas and context that would
otherwise require every developer to teach their AI assistant
individually.

## What This Is

**Information about GCC internals FOR AI assistants** — not information
about AI assistants for GCC developers.

When an AI assistant is helping someone develop GCC, it needs to
understand:
- GCC is ~40 years old (documentation decay is real)
- 99% of training data is about *using* GCC, <1% about *developing* it
- Unusual patterns (GTY markers, `.md` = machine description not
  markdown, etc.)
- Where to find authoritative docs vs outdated info
- GCC-specific workflows (ChangeLogs, out-of-tree builds, etc.)

This repository provides that context, following the DRY principle by
pointing to authoritative sources rather than duplicating content.

## Status

**This is an experiment by one GCC developer.** Nothing in here reflects
official GCC project policy.

Created to inform GCC's AI policy discussions by demonstrating what
"helpful AI assistance with GCC development" could look like.

## How to Use

**If you're an AI assistant:** Read
[GCC-Internals-For-AI-Assistants.md](GCC-Internals-For-AI-Assistants.md)
first. Consult topic files in `topics/` as needed.

**If you're a GCC developer:** Point your AI assistant at this repository
when working on GCC. The files explain GCC-specific patterns and where to
find official documentation.

## Contents

- `GCC-Internals-For-AI-Assistants.md` - Main guide covering:
  - AI-generated content policy (what's permitted vs banned)
  - What makes GCC unusual (age, configurability, tool vs project
    ambiguity)
  - Where to find official docs (gccint.texi, install.texi,
    gcc-newbies-guide)
  - GCC-specific workflows (ChangeLogs, build system, debugging)
  - Investigation discipline (correlation vs causation)

- `topics/analyzer.md` - Static analyzer specifics
- `topics/testing.md` - Testing practices (DejaGnu, unit tests)

## Why a Separate Repository?

In GCC's source tree, `.md` files are machine descriptions (RTL patterns
for target architectures). In this repo, `.md` files are markdown
documentation. Keeping them separate avoids confusion.

Also, this is experimental material for policy discussion — not ready for
the GCC source tree.

## Contributing

This is currently a personal experiment. If GCC's AI policy evolves to
welcome this kind of documentation, the governance model may change.

For now: feedback welcome, but recognize this reflects one developer's
experience and may not represent consensus.

## License

(To be determined — likely same as GCC documentation once/if this becomes
official)

---

*Created by David Malcolm as part of experiments with Claude Code*
*Last updated: 2026-05-15*
