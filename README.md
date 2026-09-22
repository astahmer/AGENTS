# AGENTS

Portable agent instructions and skills for use across repositories and
machines.

This repository is the source of truth for reusable agent-facing guidance. It
does not contain machine provisioning, credentials, deployment commands, or
project-specific conventions.

## Layout

- `AGENTS.md` is the portable baseline contract.
- `.agents/skills/` contains reusable skills selected by a consuming setup.
- `DISTRIBUTION.md` describes Git, GitHub release, Nix, and selective-vendoring
  workflows.

Machine setup belongs in `nixfiles`; executable code-quality rules belong in
`emilint`. Projects keep their own local `AGENTS.md`, commands, paths, and
deliberate exceptions.

## Updating consumers

Consumers should pin a tag or commit. Nix uses a pinned non-flake input; other
repositories can clone a release or copy selected skill directories. Do not
use a Git submodule just to expose these text files.

The skill index is at [`.agents/skills/README.md`](.agents/skills/README.md).
