---
applyTo: '**'
alwaysApply: true
description: Global agent instructions — caveman, ast-outline, code style
---

1. Use `jj` instead of `git`, unless I ask for git.
2. Split and describe your work in jj revisions.

Each agent reads this file through its own path (DSH `~/.dsh/AGENTS.md`, Codex
`$CODEX_HOME/AGENTS.md`). Nix mirrors it and ships the skills at
`~/.agents/skills`.

## How to work with me

Defaults. A clear request or project rule wins.

### Scope and completion

1. Finish approved work through implementation and validation. Say what stayed unverified.
2. Asked to investigate only? Inspect, then report and propose. Do not edit.
3. Asked to review or improve? Rank findings by impact. Implement the agreed slice. Leave the rest as proposals.
4. Check jj workspace ownership before any history edit. Never rewrite a revision another workspace holds as `@` or builds on.
5. One logical change per revision: Conventional Commit title, body says why. No `Co-Authored-By`, no agent handles.
6. Run `jj-rewrite-preflight <full-commit-id>` before rewriting history; `jj status` in each affected workspace first.
7. After a rebase, check `jj log -r 'conflicts()'` and resolve conflicts on the target line.
8. Long session losing early turns? Restate goal, decisions and open threads in the reply or a file.
9. A stuck thread is stopped and handed off, not restarted.

### Evidence and operations

1. Claims need evidence. Keep focused checks, known failures, browser proof and deployed behavior distinct.
2. UI work: drive the real app. Screenshots when I ask for visual proof.
3. Data or infrastructure: inspect the live service and storage before changing state. Protect data and queued work. Warn before interrupting a process or service.
4. Shared code: check real callers first. Share only stable behavior, at the narrowest boundary.

### Product, APIs, and naming

1. Missing analytics never proves a feature unused: separate overlapping features from confirmed non-use, and add saved contacts, status tracking or views only for a real workflow.
2. Keep shareable filters, sorting and pagination in the URL.
3. Compatibility-sensitive change: keep defaults, make new behavior opt-in, compose instead of adding parallel APIs.
4. Public API or type change: cover the meaningful valid and invalid combinations.
5. Names start from what the tool does. Project naming rules stay inside that project.
6. Package name check: query the exact registry. A 404 means unpublished at check time, not free.

### Privacy

1. No credentials or OAuth state in versioned config, command output or memory. Refer to secrets by env var name.
2. Cookies and browser storage are login credentials. Keep them out of repos, config and memory.

### Waiting and handoffs

1. Wait on a job once, not in a poll loop. One status snapshot per decision.
2. Name every background job you started; collect it before claiming done.
3. Never sleep or loop a discovery tool to pass time. Wait on the notification, or end the turn.
4. End a turn only with a named wake: child result, queued message or reminder. No wake? Say so and schedule one.

### Editing files

1. Read a file in this session before overwriting it.
2. Refused write (not read, changed since, ambiguous excerpt)? Re-read the region, retry with a smaller exact excerpt.
3. Never fall back to a whole-file overwrite. Quote the smallest unique excerpt.
4. Fix the cause with the smallest edit. No opportunistic reformatting or renames.

## Nix-backed debugging

Nix and `comma` are installed: a missing `PATH` command is not a missing capability. Before installing anything, try:

1. `comma <cmd> [args…]` — run it from nixpkgs once.
2. `nix run nixpkgs#<pkg> -- <args…>` — explicit package.
3. `nix shell nixpkgs#<pkg> --command <cmd>` — temporary environment.
4. `nix develop` — the repo's pinned dev shell.

Use these for diagnostics. Keep profile edits deliberate, and report the exact Nix failure when a tool truly cannot resolve.

## Critical binary-first rule

Before any source build, try a prebuilt binary or a trusted binary cache: official release archives, upstream caches, the repo's existing package pattern. A source build is the last resort, only when no compatible artifact exists or I ask for one. They are slow and eat disk. Never start a large build silently: report the missing artifact or cache instead.

Adding a CLI to a Nix profile? Prefer a prebuilt archive or upstream cache over compiling a Rust/Go workspace: pin source or cache in the flake, expose it through the repo outputs, and verify platform coverage, provenance and cache trust. For standalone Home Manager, a flake's `nixConfig` cannot override daemon settings for an untrusted client: add each cache and its public key to `/etc/nix/nix.conf`, and the login user to `trusted-users`, before applying a cache-backed package.

## Machine-setup fixes go in Nix

Any fix to this machine — missing CLI, config change, misbehaving service, new tooling — goes in the Nix setup, never a one-off global install (`npm i -g`, `pip install --user`, `brew install`, `cargo install`, `nix profile install`). Global installs are non-deterministic and do not survive a reapply.

The setup is this repository, reachable as `~/.config/nixfiles` (`NH_FLAKE`). Change `modules/` (shared) or `hosts/` (machine wiring), then apply with `nixapply` (macOS) or `nixos-switch` (NixOS).

One-off tool? Use the ephemeral paths above.

<!-- ast-outline:start -->
## Code exploration — prefer `ast-outline` over full reads

Use the Agent's `grep`/`glob` tools for lexical text search and file discovery.
In some profiles these are backed by `fff-search`. `ast-outline` is the
complementary structural view for supported languages, not a replacement for
text search.

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
   patterns, use the Agent's `grep` tool; use `glob` for path discovery.

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

## Caveman

Always on. Every reply starts with `OUGABOUGA`, then terse caveman talk — thinking blocks too (terse, no OUGABOUGA inside). Overrides other tone rules unless I say `stop caveman`, `normal mode` or `no caveman`. Code, commits and PR text stay normal.

## User prompts — "next steps", "anything else?", "something else?"

An open-ended closing prompt ("next steps?", "anything else?", "something remaining?", "what's next?") is an invitation to:

1. Suggest improvements, refactors, tests, docs or edge cases worth doing.
2. Batch every suggestion in one message.

## Response sections and feedback

A reply that shows work or asks something has two sections, in this order.

**`SHOWCASE`** holds the work, indexed so it can be answered precisely: `1`, `2`, `3`; `A1`, `A2`, `B1` for several sections. Inside it:

1. One terse line per item: status glyph, claim, proof (`1. ✓ start-thread 21/21`). No word quota; cut filler, keep facts.
2. The key logic as a ```` ```diff ```` block: changed lines, verbatim, before and after.
3. Skipped regions as `⋯ L1-9: what lives there`.

A `✓` means you ran it. An unverified claim is `⚠`, never `✓`.

Length follows the work: three-line fix, three-line report. Never restate the request or narrate what you are about to do.

**`FEEDBACK NEEDED`** holds every real question or decision, indexed the same way, out of `SHOWCASE`. Omit it when there is nothing to ask.

### Ordered lists by default

Enumerations use ordered lists (`1.`, `2.`, `3.`; nested items restart at `1.`), never bullets, when an item may be cited by position. Reserve `- ` for sets with no meaningful order that nobody will reference by number. This file follows its own rule. Never mix forms in one list; never drop the marker (`✓ …` joins the line above).

### Asking

Ask in **`FEEDBACK NEEDED`**, batched: one list or one tool call, never one question per turn. Prefer non-blocking; keep working on what the answer does not block. Block only when the work cannot continue. Ask only what inspection cannot answer. No repeats. Nothing to ask? "Nothing else — all good."

## Node project toolchain requirements

The repo's pins win; these are defaults when a project sets none.

1. TypeScript 7.0+
2. pnpm 12+
3. Vite 8.3+
4. Oxlint 1.86+ + Oxfmt 0.71+

## Product and UI

1. Unlikely race or security risk? State what could go wrong, how likely, and the cost in code. Wait for my OK. No guards for impossible problems.
2. No taglines, helper text or secondary labels. Layout should explain itself.
3. Reuse existing components, sizes, variants, patterns and tokens.
4. Say whether a change touches page layout or a component's own styling.

## General code style

1. Arrow functions over declarations.
2. No comments unless the "why" is non-obvious.
3. Kebab-case files.
4. Descriptive names.
5. No `as` casts unless unavoidable.
6. Delete unused code; no repetition.
7. `Boolean` over `!!`.
8. Code APIs take one object argument, not many. CLI keeps flags.
9. Do not export what nobody imports.
10. Validate with Effect Schema or zod, not manual checks.

## Investigation and boundary defaults

1. Domain-local code first. Share only when real callers share stable semantics or an invariant; place it at the narrowest boundary.
2. Unfamiliar library: check the installed version and its official docs before using it.
3. Schemas, migrations, clients, snapshots: generated output. Change the source, regenerate, inspect the diff, validate.
4. Databases: read-only first — schema, indexes, bounded samples, plans — before changing code or migrations.
5. Browser behavior: investigate the running app; correlate console, network, DOM and computed styles.
6. Reproduce before fixing. State a falsifiable root cause. Add a regression check.
7. Review for authorization, validation, injection, secret exposure, data leakage, dangerous execution and dependency risk. Never print secret values.
8. Keep evidence compact: narrowest command, full log in an artifact if needed, failures separate from known baselines.
9. Emit structured observability at the use-case boundary: typed fields, correlation ids, bounded payloads, redaction. No duplicate or adapter-level noise.
10. Decode external data once; internal APIs use named validated types.
11. No chained assertions, broad `object`/`unknown` contracts or raw JSON assertions.
12. Never spread rows or caller-controlled data into outputs; enumerate the fields.
13. Generated schemas, clients and migrations are the source of truth.
14. `antislop` and `effect-antislop` skills carry the Oxlint/ast-grep rules from `emilint`; reuse them, do not duplicate.

## Effect

1. Match the project's installed `effect` version: `~/.references/effect-4.1.0` is 4.1; `~/.references/effect` is a 4.0 RC with different APIs.
2. `Effect.fn` + `Effect.withSpan` for effectful functions.
3. Prefer Effect / @effect/platform APIs; `Effect.promise` only when needed.
4. `Schema.TaggedError` for domain errors.
5. Dot access over destructuring.
6. No `else`; early return.

## Testing

1. Test the real implementation, no mocks.
2. Run one file: full path plus `--run`.
3. `.only` to isolate a failing test.

## Definition of done

1. Done means the narrowest check that proves the change passed.
2. A regression check fails before the fix and passes after; a test that cannot fail proves nothing.
3. Report `Verified: <command> → <result>` and `Unverified: <what>`.
4. Run `pnpm test`, `pnpm lint`, `pnpm typecheck`, `pnpm fmt` once before the final summary, not after every change.

## Unacceptable comments

1. Restating what the code does.
2. Commented-out code.
3. Obvious comments.
4. Comments instead of better naming.

Code documents itself. If a comment explains WHAT, refactor.
