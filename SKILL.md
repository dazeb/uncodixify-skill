---
name: uncodixify
description: >-
  Use when React or Tailwind components look too generic, dashboard-like, or
  AI-template-ish (gradients, glassmorphism, oversized radii, purple accents).
  Audits component styling against strict design constraints, flags banned CSS
  patterns, and enforces a dark restrained aesthetic via an automated validator
  that exits 0 on pass. Trigger phrases include UI looks generic, too much like
  a dashboard, fix the styling, Tailwind design system, component looks like a
  template, clean up the CSS.
---

# Uncodixify

## Overview

Use this skill when a frontend task needs strict anti-template UI constraints and an explicit audit loop. The goal is not "fancier" UI. The goal is restrained, product-shaped UI that avoids generic AI-dashboard tropes.

This package is intentionally isolated for eval work. Treat it as an eval target, not as part of the shared `.agents/skills` catalog.

## When to Use

- React or Tailwind work is trending toward gradients, glassmorphism, oversized radii, or decorative dashboard filler.
- A task needs a repeatable check against banned UI patterns before completion.
- You want a with-skill vs baseline eval loop for frontend styling behavior.

Do not use this skill as a general design system replacement. It is a narrow constraint package.

## Core Constraints

Before running the audit loop, keep these rules intact:

1. Use Tailwind v4 patterns.
2. Prefer dark, restrained base surfaces such as `bg-zinc-950`, `bg-black`, or `bg-neutral-900`.
3. Do not use purple accent classes.
4. Keep corner radii small: `rounded-sm`, `rounded-md`, or none.
5. Use subtle separators such as `border-white/10` or `divide-white/10`.
6. Ban glassmorphism, gradients, dramatic shadows, floating panels, and eyebrow-label copy.

## Audit Loop

1. Generate or edit the component.
2. Run `./bin/run-uncodixify.sh <path-to-file>`.
3. If the validator fails, fix the specific issues it reports.
4. Repeat until the command exits with code `0`.

The wrapper script resolves the package paths for you. Do not assume the current working directory is the package directory.

### Example: Validator Failure and Fix

Banned pattern (fails validation):
```tsx
// BEFORE — violates gradient and glassmorphism rules
<div className="bg-gradient-to-br from-purple-900 to-indigo-800 backdrop-blur-md rounded-2xl shadow-2xl">
```

Compliant replacement (passes validation):
```tsx
// AFTER — restrained dark surface, small radius, no gradient
<div className="bg-zinc-950 rounded-sm border border-white/10">
```

Typical validator output on failure:
```
FAIL src/components/Card.tsx
  [banned-gradient] bg-gradient-to-br — gradients are not allowed
  [banned-radius]   rounded-2xl — use rounded-sm or rounded-md
Exit code: 1
```

## Eval Iterations

Use `python3 ./bin/run-evals.py` to scaffold an eval iteration under `workspace/iteration-N/`.

Each eval case gets:

- `with_skill/` for the package-guided run
- `without_skill/` for the baseline run
- `prompt.txt`, `inputs/`, `outputs/`, `timing.json`, and `grading.json`

After runs are complete, aggregate the benchmark with:

`python3 ./bin/run-evals.py --aggregate-only --iteration-dir <absolute-iteration-path>`

To grade mechanical assertions in the generated `outputs/` directories and refresh `benchmark.json` in one step, run:

`python3 ./bin/grade-evals.py --iteration-dir <absolute-iteration-path>`

To run the package end to end against its bundled fixtures, use:

`python3 ./bin/run-package-evals.py`

## Package Files

- `manifesto.md`: expanded design rules and banned patterns.
- `evals/evals.json`: starter eval cases for iteration work.
- `toolchain/`: package-local evaluator implementation.
- `workspace/`: ignored eval outputs, iteration folders, and review artifacts.
