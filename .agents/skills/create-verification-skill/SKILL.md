---
name: create-verification-skill
description: Create a project-local verification skill (verify-<app>) that lets agents launch, check, drive, capture evidence from and clean up the real app on this machine through a small scripted CLI and a feature map. Use when a project has no reliable way for an agent to prove its own changes end to end, when agents keep hand-building the same dev environment, or when asked to create a verification or control-app skill.
---

# Create a verification skill

The goal is a closed loop: an agent changes the app, runs it the way a user
would, and proves the result without a human checking. The output is a
project-owned skill, `.agents/skills/verify-<app>/`, plus the tooling it calls.

## Principles

- **Build the lever.** Prefer commands over prose. Every step an agent would
  otherwise improvise (start, seed, log in, navigate, capture, stop) becomes a
  script or CLI subcommand in the repository.
- **Unexecuted means draft.** The skill is done only after you ran every
  command in it end to end in this session.
- **Real user path.** Drive the UI, API or CLI that users use. No test-only
  endpoints or shortcuts that skip the behavior being verified.
- **Isolated and owned.** Use a scratch instance with its own data directory
  and port. Never touch the user's running instance or real data. Stop only
  the processes you started.
- **Evidence stays.** Screenshots, traces, logs and command output go to a
  known directory and are reported by path.

## Process

1. **Inventory what exists.** Read the README, dev scripts, e2e fixtures,
   seed data, test helpers and any feature catalog. Reuse them: the CLI should
   wrap existing fixtures, not fork them.
2. **Choose the runtime handle.** Browser apps: Playwright or CDP. Desktop:
   the app's debug protocol or accessibility tree. iOS/Android: the simulator.
   CLIs/services: the process plus its logs and API. If no handle exists,
   build a small one.
3. **Build the CLI.** Subcommands, at minimum:
   - `up`: start a scratch instance (own home/data dir, free port, seeded
     data, test account). Record pid, URL and paths in a state file.
   - `doctor`: read-only health check with actionable failures.
   - `open <feature>`: drive to a feature from the feature map.
   - `shot` or `evidence`: capture screenshot, console/log errors, trace.
   - `down`: stop what `up` started and remove its scratch data; `--dry-run`.
   CLI rules: JSON output with `--json`, rich `--help`, `--dry-run` on
   anything destructive, and errors that name the fix ("port busy: pass
   --port or run `down`").
4. **Feature map.** If the project already has a machine-checked feature
   catalog, add a short "how to reach it as a user" field there. Otherwise
   create `references/features/README.md` (one line per feature, linked) and
   one file per feature: what it does, how to reach it, how to drive it,
   gotchas. One source of truth; never two maps.
5. **Write the SKILL.md.** Sections: When to use, Launch, Doctor, Drive,
   Evidence, Cleanup, Gotchas, Safety. Each section is mostly commands. Under
   100 lines; push details into the feature map.
6. **Prove it.** Run up → doctor → open two or three real features → evidence
   → down. Confirm nothing is left running and the scratch data is gone. Fix
   the CLI or the skill until this loop passes without improvisation.

## Safety

- Credentials come from the environment or a secret manager; scratch copies
  are mode 600 and deleted on `down`. Never print tokens, keys or
  authenticated URLs.
- Never trigger native OS dialogs (file pickers, keychain prompts) that land
  on the user's screen; register data through config or the app's API.
- Under machine load, run one instance at a time.

## Gotchas

- Register paths by their canonical form (`realpath`); on macOS `/var/...`
  and `/private/var/...` differ and the app may not match them.
- Keep one browser process alive across CLI calls (record its pid in the
  state file); a new connection per command loses the page `open` reached.
- Wait for async UI (filtered lists, pickers) with an `expect` before acting.
- Seed state through the user path when shortcuts produce a different state
  than users get.

## Report

The created files, the proof run (commands, key output, evidence paths), and
any step that still needs a human.
