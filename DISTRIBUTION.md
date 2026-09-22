# Distribution

The repository is deliberately a plain Git source tree. Choose the smallest
distribution mechanism that fits the consumer.

## Checkout

For a reusable local checkout, pin a release tag or commit:

```bash
jj git clone https://github.com/astahmer/agents /path/to/agents
```

Keep the checkout read-only from consuming projects. Copy or link only the
selected `AGENTS.md` and `.agents/skills/<name>/` files into a project-owned
agent tree.

## GitHub release

After releases are created, download a pinned source archive with GitHub CLI:

```bash
gh release download v<version> --repo astahmer/agents --archive=tar.gz
```

`gh` is the transport layer; the repository layout remains the contract.

## Selective checkout

When only one skill is needed and a full checkout is undesirable, use Git's
partial clone and sparse checkout:

```bash
git clone --filter=blob:none --no-checkout \
  https://github.com/astahmer/agents /tmp/agents
git -C /tmp/agents sparse-checkout set \
  .agents/skills/focused-validation
git -C /tmp/agents checkout <tag-or-commit>
```

This is a convenience for extraction, not a replacement for pinning the source
version in the consuming repository.

## Nix

Nixfiles should consume a pinned non-flake input and assemble the fetched tree
with machine-local overlays:

```nix
agents = {
  url = "github:astahmer/agents?ref=v<version>";
  flake = false;
};
```

The Nix module owns deployment to `~/.agents`; this repository owns the
portable source content.

## Boundaries

- Portable agent contract and general skills: this repository.
- Oxlint and ast-grep implementations: `emilint`.
- Machine-specific skills, secrets, MCP, and activation: `nixfiles`.
- Project commands, paths, and exceptions: the project repository.

Avoid Git submodules. They pin nested history but add clone, update, and
workspace friction without improving the source-of-truth boundary.
