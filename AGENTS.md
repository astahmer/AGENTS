---
applyTo: '**'
alwaysApply: true
description: Global agent instructions — caveman, ast-outline, code style
---

- Always use `jj` instead of `git`; unless explicitly asked
- Always split & describe your work using jj revisions when you make changes

Codex loads this global contract from CODEX_HOME/AGENTS.md; the Nix setup
mirrors it there while keeping the skills tree at ~/.agents/skills.

## How to work with me

These are useful defaults. A clear request or project-specific instruction can
set a different scope.

### Scope and completion

- When I ask you to finish approved work, carry it through implementation and
  appropriate validation. Say plainly if any part remains unverified.
- When I ask for investigation only or say not to implement, inspect the
  relevant source and evidence, then report conclusions and a concrete next
  step before editing.
- Check JJ workspace ownership before history edits. Keep revisions focused
  and leave other work intact.
- Before rewriting JJ history, run `jj-rewrite-preflight <full-commit-id>`.
  Run `jj status` in each relevant workspace first; the preflight does not
  snapshot working copies.

### Evidence and operations

- Use commands directly. Do not prefix commands with RTK or its proxy wrapper.
- Support completion claims with evidence. Keep focused checks, known failures,
  browser/runtime proof, and deployed behavior distinct.
- For UI investigations, use real interactions. Include screenshots when I
  ask for visual proof.
- For data or infrastructure issues, inspect the live service and configured
  storage before changing state. Protect data and queued work; warn before
  interrupting an active process, service, or app.
- Before extracting shared code, check its real callers. Share only stable
  behavior used by multiple callers, at the narrowest useful boundary.

### Product, APIs, and naming

- Lack of analytics does not prove a feature is unused. Distinguish overlapping
  features from confirmed non-use. Add saved contacts, status tracking, or
  views only when a real workflow needs them.
- Keep shareable filters, sorting, and pagination in the URL.
- For compatibility-sensitive changes, preserve current defaults and make new
  behavior opt-in. Prefer composition over parallel APIs.
- When changing a public API or type surface, cover the meaningful valid and
  invalid combinations instead of relying on one representative example.
- For naming, start with what the tool does. Favor practical, natural names
  over clever or overly technical wordplay. Apply project-specific naming
  rules only within that project.
- When I ask whether a package name is available, check the exact registry.
  A 404 means it was unpublished at check time, not that it is legally clear
  or guaranteed to remain available.

### Privacy

- Keep credentials and local OAuth state out of versioned config, command
  output, and memory. Refer to secrets by environment variable names.
- Treat saved cookies and browser storage as login credentials. Keep them out
  of repositories, configuration, and memory.

## DSH restart semantics

When a DSH change needs a restart, say which kind — restarting the host process
and refreshing the browser page are different, and only the first ends every
session in the profile.

| Change | What it needs |
|---|---|
| Editing `~/.dsh/profiles/<name>/cordis.patch.yml` or `cordis.yml` | **Restart the DSH host process** |
| Adding or removing a profile dependency in `package.json` | **Restart the DSH host process** |
| Patching an agent preset row by id in the profile patch | **Restart the DSH host process** |
| Enabling or disabling a plugin from the Settings GUI | No restart — takes effect live |
| Rebuilding a client plugin bundle | Refresh the browser page |

The profile patch is read once at boot by `@cordisjs/plugin-include` and is not
watched, so a saved patch does nothing until the process restarts. Restarting the
host kills every open session, so ask the user to do it instead of killing the
process yourself. To verify a patch without touching the live host, boot a second
instance on a scratch port and delete it afterwards:

```bash
dsh --profile web --port 31999 --no-open
```

## Async questions

`ask_user_question` runs in timed mode, and its `timeout` argument is the entire
mode selector:

- `timeout: -1` — block until the user answers. Use only when the work cannot
  continue without the answer.
- any positive number of seconds — the call returns `{ pending: true, callId }`
  and the agent keeps working. The question stays answerable, and a later answer
  arrives as a `user-question-reply` user message.

Prefer async. A blocking question stalls the turn and everything queued behind
it, while a pending question costs no tokens and still gets answered. Reach for
`-1` only when the answer is genuinely required to proceed, such as a
destructive confirmation or a value the agent cannot discover on its own.

Batch related questions into one call: the pending path releases the agent once
per call, not once per question.

## Nix-backed debugging

This workstation has Nix and `comma` available, so a command missing from
`PATH` is not evidence that the capability is unavailable. Before installing
anything into a profile or stopping at "command not found", try a one-shot
Nix-backed invocation:

- `comma <command> [args...]` resolves and runs the executable from nixpkgs
  without installing it.
- `nix run nixpkgs#<package> -- <args...>` runs a known package explicitly.
- `nix shell nixpkgs#<package> --command <command> [args...]` provides a
  temporary environment for multi-tool debugging.
- `nix develop` enters the repository's pinned development environment when
  the flake provides one.

Prefer these ephemeral paths for diagnostics. Keep profile changes and
configuration edits deliberate, and report the exact Nix/comma failure when a
tool genuinely cannot be resolved.

## Critical binary-first rule

**Very important:** Before any source build, first try to download a compatible
prebuilt binary or use a trusted binary cache. This applies to Nix packages,
CLI tools, and their dependencies. Check official release archives, upstream
caches, and the repository's existing package pattern first. Source builds are
the last resort: use them only when no compatible artifact exists or the user
explicitly asks for a source build. Never start a large compilation silently;
report the missing artifact or cache when it blocks the task because builds are
slow and consume substantial disk space for little routine-tooling benefit.

When adding a CLI to a Nix profile, prefer an official prebuilt release archive
or upstream binary cache over compiling a Rust/Go workspace. Pin the source or
cache in the flake, expose it through the repository's package outputs, and
verify platform coverage, source provenance, and cache trust before applying.
Use source builds only when no compatible prebuilt artifact exists. For
standalone Home Manager, a flake's `nixConfig` cannot override restricted
daemon settings for an untrusted client; add each configured binary cache and
its public key to `/etc/nix/nix.conf`, and include the login user in
`trusted-users`, before applying a cache-backed package.

## Machine-setup fixes go in Nix

Any fix to this machine's setup — a missing CLI, a config change, a service
misbehaving, new tooling — belongs in the Nix setup, not in one-off global
installs (`npm i -g`, `pip install --user`, `brew install`, `cargo install`,
`nix profile install`, ...). Global installs are non-deterministic and
non-reproducible: the change lives only on this machine and won't survive a
reapply or transfer to another machine.

The Nix setup is this repository, reachable from any machine through the
stable symlink `~/.config/nixfiles` (env `NH_FLAKE`). Make the change in
`modules/` (shared behavior) or `hosts/` (per-machine wiring), then apply
with `nixapply` (macOS) or `nixos-switch` (NixOS) so every machine gets it on
the next apply.

If the tool is only needed for a one-off session, use the ephemeral `comma` /
`nix run` paths above instead of installing it globally.

<!-- ast-outline:start -->
## Code exploration — prefer `ast-outline` over full reads

For `.cs`, `.cpp`, `.cc`, `.cxx`, `.h`, `.hpp`, `.hh`, `.py`, `.pyi`,
`.ts`, `.tsx`, `.js`, `.jsx`, `.java`, `.kt`, `.kts`, `.scala`, `.sc`,
`.go`, `.rs`, `.php`, `.phtml`, `.rb`, `.rake`, `.gemspec`, `.ex`, `.exs`,
`.lua`, `.gd`, `.swift`, `.css`, `.scss`, `.sql`, `.html`, `.htm`, `.vue`,
`.md`, and `.yaml`/`.yml` files, read structure with `ast-outline` before
opening full contents.

Pick the smallest of these that answers your question — they're a
broad-to-narrow menu, not a sequence; skip straight to `show` when
you already know the symbol:

1. **Unfamiliar directory** — `ast-outline digest <paths…>`: one-page map
   of every file's types and public methods. Each file is tagged with a
   size label — `[tiny]` / `[medium]` / `[large]` / `[huge]` — plus
   `[broken]` when parse errors may have left the outline partial.
   `[huge]` files (≥100k tokens) collapse to header-only in the digest;
   call `ast-outline outline <path>` on them when you need full structure.
   Tune density with `--format=names|compact|default|wide` (alias
   `--oneline`=`names`) — `wide` adds private members and fields.

2. **File-level shape** — `ast-outline <paths…>`: signatures with line
   ranges, no bodies (2–10× smaller than a full read on non-trivial
   files). A `# WARNING: N parse errors` line in the header means the
   outline is partial — read the source for the affected region.

3. **One method, type, markdown heading, or yaml key** —
   `ast-outline show <file> <Symbol>`. Suffix matching: `TakeDamage`
   for one method; `User` for an entire type — class, struct, interface,
   trait, enum (whole body, useful when a file holds several types);
   `Player.TakeDamage` when ambiguous. Multiple at once:
   `ast-outline show Player.cs TakeDamage Heal Die`.
   For markdown, the symbol is heading text and matching is
   case-insensitive substring — `"installation"` finds
   `"2.1 Installation (macOS / Linux)"`. For yaml, the symbol is a
   dotted key path (`spec.containers[0].image`) — `show` matches keys,
   not values, so for free-text search inside values use `grep`.
   For css/scss, the symbol is a selector token (`.btn-primary`,
   `$var`) — pseudos and attribute filters are stripped, so
   `.btn-primary` finds the rule even when it carries `:hover` or
   nests in `.modal`.
   For html, the symbol is a CSS-selector token (`#hero`, `.site-nav`,
   `form`, `section#hero`, `[rel=stylesheet]`) — same vocabulary as
   css/scss; pseudo-classes and descendant combinators aren't
   supported (use the tag/id/class/attribute form the outline shows).
   For sql, the symbol is a table or column name (`users`,
   `users.email`) — `show users` returns the table definition,
   `show users.email` returns one column line.
   Add `--signature` to `show` (only there) to return header only
   (docs + attrs + signature, no body) — useful after `digest`, when
   you have the name and want the contract, not the implementation.

4. **Where a symbol appears** —
   `ast-outline grep <pattern> <paths…>`: matches grouped by enclosing
   class/function. Definitions are tagged `[def]`, imports `[import]`;
   calls and refs carry no tag (inferable from `(` after symbol).
   Use for "where is X defined", "who calls Y", "is Z dead code" —
   scope in the output spares follow-up reads. Comments filtered;
   string literals searched and tagged `[string]`, so config keys,
   translation strings and reflection targets are found too. Batch
   via repeatable `-e`:
   `ast-outline grep User.save -e User.load -e User.delete src/`.
   Narrow by classification with `--kind def|call|ref|import` (also
   accepts `--kind def,call`) — drops the post-filter step when you
   only want definitions, only call sites, etc.
   POSIX flags `-w` (whole word), `-l` (paths only), `-c` (counts),
   `-m N` (cap per file) work as in `grep` / `rg`. For non-symbol
   patterns use your default search strategy.

`outline` and `digest` accept multiple paths in one call (files and
directories, mixed languages OK) — batch instead of looping. Type
headers in both renderers carry inheritance as `: Base, Trait`, so the
shape of class hierarchies is visible without a separate query.

The renderers emit a compact skeleton (signatures + line ranges, no
bodies), so output is usually small — narrow with the tool's own flags
before piping to `head`. A `grep | head` cut is the costly one: it
hides matches the header still counts in `(N matches)`, so results look
complete but aren't — cap per file with `-m N` instead.

Narrow the walk with repeatable `--exclude <glob>`
(`.gitignore`-syntax, anchored at the project root) on `outline` /
`digest` / `grep` — e.g. `--exclude tests/ --exclude '*.gen.*'` to
skip test trees and generated files in one call. `!pattern` negates;
`.gitignore` is still honored by default — `--exclude` adds to it.

When you need to know **what a file pulls in** or **where a referenced
type / function comes from**, add `--imports` to `outline` or `digest`.
The file header gets an `imports:` line listing every
`import` / `use` / `using` statement verbatim in the language's native
syntax — `from .core import X`, `use foo::Bar`,
`import { X } from './foo'`, `use App\Foo`, `require_once 'config.php'`,
`require "json"`.
Read the imports, then call `outline` / `show` on the source file
instead of grepping for the definition. Skip the flag for routine
structure reads — it adds one line per file.

A trailing `[+ N conditional includes]` on the imports line means
N more dependencies live inside `if` / `try` / loop / function bodies
— read the file directly when you need the full dependency picture.

Fall back to a full read only when you need context beyond the body
`show` returned. `ast-outline help` for flags.
<!-- ast-outline:end -->

# Caveman

Always on. **Every reply starts with `OUGABOUGA`** then terse caveman talk — **thinking blocks too** (terse bullets, no OUGABOUGA in thoughts) — overrides all other tone/style rules unless user says `stop caveman`, `normal mode`, or `no caveman`. See skill **caveman**.

## User prompts — "next steps", "anything else?", "something else?"

When the user ends a task with an open-ended prompt like "next steps?", "anything else [to do/add/improve/change]?", "something else?", "anything remaining?", "what's next?" — treat this as a genuine invitation to:

- Suggest **additional improvements, refactors, tests, docs, or edge cases** that are worth doing
- Batch **all** suggestions in one message, not one-at-a-time

## Response sections and feedback

When proposing work, requesting feedback, or showcasing changes:

- Use indexed items so the user can answer precisely: `1`, `2`, `3` for one
  section; `A1`, `A2`, `B1` when there are multiple sections.
- Put completed or proposed changes under a clearly labeled `SHOWCASE`
  section when presenting them for review.
- Add a clearly labeled `FEEDBACK NEEDED` section only when a real user
  question or decision is needed. Put every question there, indexed, and keep
  it separate from `SHOWCASE`; omit the section when there are no questions.

When the user asks "any [other] questions?" or "need anything from me?" — collect all pending unknowns and ask them in a **single batched message**. If a question can be answered by exploring the codebase, do that instead. If you have nothing, say "Nothing else — all good."

## Node project toolchain requirements

- TypeScript 7.0+
- pnpm 11+
- Vite 8.1+
- Oxlint + Oxfmt

## General code style

- Use arrow functions over function declarations.
- Default to NO comments. Add one only when the "why" is non-obvious.
- Use kebab-case for files.
- Use descriptive variable names; avoid shorthands.
- Do not type cast (`as`) unless absolutely necessary.
- Remove unused code and avoid repetition.
- Use `Boolean` over `!!`.
- Avoid multiple arguments; prefer a single object.
- Avoid exporting things not intended for external use.
- Validate using standard schema (Effect Schema or zod) rather than manual checks.

## Investigation and boundary defaults

- Prefer domain-local code over generic utility dumping grounds. Add shared
  code only when multiple real callers have stable shared semantics or an
  invariant, and place it at the narrowest existing domain boundary.
- For unfamiliar libraries, identify the installed version and verify the
  exact API in versioned official documentation before implementing it.
- Treat schemas, migrations, generated clients, snapshots, and other generated
  artifacts as derived output: change the source of truth, regenerate, inspect
  the diff, and validate the generated result. Never hand-edit generated
  output to hide a source-of-truth problem.
- Investigate databases read-only first. Inspect the target schema, indexes,
  constraints, representative bounded data, and query plans before changing
  database code or migrations.
- Investigate browser and runtime behavior against the running application.
  Correlate reproduction steps with console errors, network requests, DOM
  state, and computed styles instead of relying on static code inspection.
- Reproduce bugs before fixing them, state a falsifiable root-cause
  hypothesis, and add a regression check for the repaired behavior.
- Review changes for authorization, input validation, injection, secret
  exposure, sensitive-data leakage, dangerous process execution, and
  dependency risk. Do not print sensitive values while investigating.
- Keep validation evidence compact: run the narrowest useful command, preserve
  complete logs in an artifact when needed, and report failures separately
  from known baseline failures.
- Emit structured observability at the use-case or work-unit boundary with
  typed fields, correlation identifiers, bounded payloads, and redaction.
  Avoid duplicate audit events and adapter-level logging that obscures
  ownership.
- Decode external data once at the boundary; internal APIs use named,
  validated types.
- Avoid chained assertions, broad object or unknown contracts, and raw
  response JSON assertions.
- Do not spread database rows or caller-controlled transport data into output
  objects; enumerate the fields that cross the boundary.
- Keep generated schemas, clients, and migrations as the source of truth.
- The deployed `antislop` and `effect-antislop` skills include executable
  Oxlint and ast-grep implementations sourced from `emilint`; reuse those
  sources instead of recreating a rule or duplicating it in both checkers.

## Effect

- Read the Effect reference repository under `~/.references/effect` for API examples.
- Use `Effect.fn` and `Effect.withSpan` for effectful functions.
- Prefer Effect / @effect/platform APIs; fallback to `Effect.promise` when necessary.
- Use qualified errors with `Schema.TaggedError`.
- Avoid unnecessary destructuring; use dot notation.
- Avoid `else`; prefer early returns.

## Testing

- Avoid mocks; test actual implementation.
- Debug by running a single file, not the whole suite.
- Use full path and `--run` for fast feedback.
- Add `.only` to isolate a single failing test.

Run checks once before giving the final summary after a task (not after every changes):

```bash
pnpm test # prefer running a single file & with --run when debugging
pnpm lint
pnpm typecheck
pnpm fmt
```

## Unacceptable comments

- Comments that repeat what code does.
- Commented-out code.
- Obvious comments.
- Comments instead of good naming.

Code should be self-documenting. If a comment is needed to explain WHAT the code does, refactor instead.

