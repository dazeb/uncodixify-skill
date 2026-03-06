# Uncodixify

Portable skill package for constraining React and Tailwind UI away from generic AI-dashboard styling.

This package is designed to be copied between projects as a single folder. It includes:

- the skill prompt in `SKILL.md`
- the longer design reference in `manifesto.md`
- a deterministic validator in `toolchain/uncodixify.ts`
- a deterministic autofix pass in `toolchain/autofix-uncodixify.ts`
- isolated eval fixtures and assertions in `evals/`
- package-local CLI entrypoints in `bin/`

## What It Enforces

The package pushes UI toward restrained, product-shaped layouts and away from common AI-generated defaults.

Core constraints:

- dark neutral surfaces
- restrained corner radius
- subtle white separators
- no purple utility classes
- no gradients or glassmorphism
- no eyebrow-label patterns like `<small>`
- no ornamental dashboard hero copy in the bundled eval scenarios

## Package Layout

```text
uncodixify/
├── SKILL.md
├── README.md
├── manifesto.md
├── package.json
├── bin/
│   ├── run-uncodixify.sh
│   ├── validate-package.py
│   ├── run-evals.py
│   ├── grade-evals.py
│   └── run-package-evals.py
├── toolchain/
│   ├── uncodixify.ts
│   └── autofix-uncodixify.ts
├── evals/
│   ├── evals.json
│   └── files/
└── workspace/
    └── .gitignore
```

`workspace/` is intentionally included but git-ignored for iteration outputs.

## Requirements

- Node.js with `npx`
- Python 3
- `tsx` available through local dependencies or `npx`

This package ships its own `package.json` with `tsx` and `typescript` as dev dependencies.

## Install

If you copy this folder into another repo, install dependencies from inside the package:

```bash
npm install
```

## Validate The Package

Run this from inside the package directory:

```bash
python3 ./bin/validate-package.py
```

This checks:

- `SKILL.md` frontmatter shape
- `evals/evals.json` schema
- assertion definitions
- workspace scaffolding

## Validate A Single File

To check one TSX file against the uncodixify rules:

```bash
./bin/run-uncodixify.sh path/to/component.tsx
```

Exit codes:

- `0`: file passes
- `1`: one or more violations were found

The validator is deterministic and regex-based. It does not rely on external APIs.

## Autofix A File

The package autofix script normalizes class tokens and removes the banned patterns it knows how to rewrite safely.

```bash
npx tsx ./toolchain/autofix-uncodixify.ts path/to/component.tsx
```

What the autofix currently does:

- rewrites oversized radius classes to `rounded-md`
- removes gradient and glassmorphism utilities
- rewrites purple utility classes to neutral alternatives
- removes uppercase tracking-eyebrow utility patterns
- replaces `<small>` with `<p>`
- injects dark base and separator classes when missing
- rewrites specific ornamental copy patterns used in the bundled eval fixtures

It is intentionally conservative. It will not perform full AST-level JSX refactors.

## Run Eval Scaffolding

To create a new isolated eval iteration:

```bash
python3 ./bin/run-evals.py
```

Or use a fixed iteration name:

```bash
python3 ./bin/run-evals.py --iteration iteration-1
```

This creates:

- `workspace/iteration-N/`
- `with_skill/` and `without_skill/` directories per eval case
- `prompt.txt`, `inputs/`, `outputs/`, `timing.json`, and `grading.json`

## Grade Existing Eval Outputs

To grade mechanical assertions for an existing iteration:

```bash
python3 ./bin/grade-evals.py --iteration-dir /absolute/path/to/workspace/iteration-N
```

This writes `grading.json` for each run and refreshes `benchmark.json`.

## Run The Full Package Eval Loop

To execute the bundled eval fixtures end to end:

```bash
python3 ./bin/run-package-evals.py
```

Or:

```bash
python3 ./bin/run-package-evals.py --iteration iteration-1
```

This flow:

1. creates an iteration
2. copies the bundled fixture files into run outputs
3. keeps the original fixture as the baseline (`without_skill`)
4. applies the deterministic autofix to the `with_skill` run
5. validates the result
6. grades all assertions
7. writes an aggregated `benchmark.json`

## Eval Design

The bundled evals are intentionally simple and mechanical. They currently test:

- file existence
- required dark-base tokens
- required separator tokens
- absence of purple classes
- absence of large radii
- absence of gradient, blur, and eyebrow-label patterns
- removal of decorative copy in the bundled scenarios
- preservation of basic form structure in the settings scenario

The aim is not to fully judge design quality. The aim is to make regressions measurable.

## Portability

This package is meant to be portable as one folder.

If you copy it into another repo, the package-local commands under `bin/` should still work as long as:

- Python 3 is available
- Node.js is available
- package dependencies are installed

The repo-level wrappers used in `x3s` are convenience proxies only. They are not required for the package to function.

## Recommended Workflow

For package development:

```bash
python3 ./bin/validate-package.py
python3 ./bin/run-package-evals.py --iteration iteration-dev
```

For applying the rules to a real component:

```bash
npx tsx ./toolchain/autofix-uncodixify.ts src/MyComponent.tsx
./bin/run-uncodixify.sh src/MyComponent.tsx
```

## Limitations

- Validation is regex-based, not AST-based.
- Autofix is heuristic and may not preserve ideal semantics for every JSX file.
- The package does not claim to evaluate holistic design quality.
- The bundled assertions are strongest for Tailwind-heavy TSX files, not arbitrary frontend stacks.

## Public Repo Notes

If this package is published as its own repository, the recommended root commands are:

```bash
npm install
python3 ./bin/validate-package.py
python3 ./bin/run-package-evals.py
```
