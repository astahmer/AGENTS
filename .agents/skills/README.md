# Skills

Reusable agent skills live here. They are intentionally product-neutral: each
consumer supplies its own commands, paths, framework choices, and policy
exceptions.

| Skill | Use when |
| --- | --- |
| `add-reference-repository` | Adding a reusable reference checkout |
| `read-reference-repository` | Inspecting a reference checkout |
| `setup-nix-in-repository` | Bootstrapping a repository's Nix and direnv shell |
| `ast-outline` | Exploring unfamiliar source code with AST-aware outlines and grep |
| `browser-debugging` | Debugging frontend behavior with browser/runtime evidence |
| `bug-investigation` | Reproducing and isolating a bug before fixing it |
| `caveman` | Applying the personal terse-response style |
| `compact-test-evidence` | Reporting focused test failures without losing diagnostics |
| `database-investigation` | Inspecting database schema, data, indexes, and plans read-only |
| `feature-plan` | Creating a structured feature plan from a reusable template |
| `grill-me` | Stress-testing a plan or design through batched questions |
| `jj` | Performing safe Jujutsu history operations |
| `library-documentation-first` | Verifying unfamiliar library APIs before implementation |
| `papercuts` | Recording short-lived, concrete workflow friction |
| `product-description` | Describing product behavior from code and runtime evidence |
| `rtk` | Using the token-optimized shell wrapper |
| `security-review` | Reviewing authorization, validation, injection, and secret exposure |
| `show-me` | Explaining a topic with focused diagrams or artifacts |
| `structured-observability` | Designing bounded, typed, correlated observability |
| `taste-from-sessions` | Maintaining durable personal coding preferences |
| `write-a-skill` | Creating or updating a reusable agent skill |
| `runtime-boundary-audit` | Reviewing ownership between UI, actors, services, adapters, and packages |
| `actor-state-ownership` | Designing actor-based frontend state and async work |
| `durable-async-work` | Deciding whether work should be inline, queued, retried, cancelled, or resumed |
| `safe-transformations` | Running codemods, migrations, generators, or source-to-source changes |
| `focused-validation` | Choosing proportionate checks and reporting evidence |
| `library-architecture` | Designing public APIs, registries, adapters, and extension points |
| `frontend-evidence-audit` | Auditing frontend behavior with static and runtime evidence |
| `generated-artifact-workflow` | Changing schemas, specifications, generated clients, snapshots, or derived files |

Keep machine-specific and product-specific skills in the consuming repository's
overlay. See [`../../DISTRIBUTION.md`](../../DISTRIBUTION.md) for reuse options.
