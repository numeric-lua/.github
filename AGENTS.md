# AGENTS.md

Canonical instructions for coding agents working anywhere in the `numeric-lua`
organization. Each repository may add local details, but these rules win.

## Workspace

This directory is an organization workspace, not a repository. The repositories
are separate Git checkouts:

| path | remote | scope |
| --- | --- | --- |
| `luatorch/` | `numeric-lua/luatorch` | tensor core, ATen FFI, autograd, fusion, `nn`/`optim` |
| `numlua/` | `numeric-lua/numlua` | NumPy-shaped arrays, `.npy`, linalg |
| `luaplotlib/` | `numeric-lua/luaplotlib` | SVG plotting |
| `tensorflow-lua/` | `numeric-lua/tensorflow-lua` | native LuaJIT binding to the TensorFlow C API |
| `lua-polars/` | `numeric-lua/lua-polars` | eager DataFrame binding over Rust Polars |
| `lua-notebooks/` | `numeric-lua/lua-notebooks` | LÖVR notebook frontend and kernels |
| `luar/` | `numeric-lua/luar` | Lua 5.5 runtime, FFI, tiered JIT |
| `numeric-lua-agents/` | `numeric-lua/numeric-lua-agents` | permanent project service and queue scheduler |
| `tensorflow-lua-addons/` | `numeric-lua/tensorflow-lua-addons` | Addons parity for tensorflow-lua |
| `.github/` | `numeric-lua/.github` | org profile and shared policy |

The repositories live in `/home/machine/Códigos/numeric-lua/`. All nine are
real checkouts; the default branch is `main` everywhere.
The organization is <https://github.com/numeric-lua>.

## Non-negotiable safety

- **Direct pushes to `main` are authorized by the human coordinator.** No pull
  request is required. The permanent project service works in the main
  checkouts and pushes tested commits; the queue scheduler uses isolated
  worktrees and may push tested agent branches for the coordinator to land.
- **No releases, tags, version bumps, package publishes, or organization-setting
  changes** unless the human coordinator explicitly asks. Workers add
  `## Unreleased` entries; the coordinator cuts releases.
- **No secrets.** API keys belong in `~/.dsh` or a mode-600 env file outside the
  repositories. Never put credentials in tasks, prompts, commits, tests, or
  screenshots.
- **One agent at a time on this machine.** The laptop has 8 cores, about 7 GB of
  RAM, and full swap. Builds and test suites are heavy. The scheduler must use
  the host-wide agent lock, and the Maria Padilha scheduler must not run
  concurrently unless it honors the same lock.
- **Do not create the GitHub organization, transfer repositories, enable Pages,
  or change repository visibility** from an agent. Those are coordinator
  actions.
- **The permanent service is coordinator-managed.** Do not install, stop,
  restart, or reconfigure its systemd unit unless the coordinator asks. The
  queue scheduler stays in `--dry-run`/`--once` until its branches have been
  reviewed.

## Architecture rules

- **One tensor/autograd engine for the numeric-lua stack.** The core is
  luatorch. `numlua` is a NumPy-shaped facade over luatorch tensors and
  `luaplotlib` plots arrays. Do not add a second FFI tensor implementation, a
  second autograd graph, or a second dtype-promotion system.
- **TensorFlow is a deliberate external runtime.** `tensorflow-lua` is a native
  LuaJIT binding to TensorFlow's C API. It does not wrap or replace luatorch,
  does not link libtorch, and is not part of the luatorch dependency graph.
- If a shared lower layer becomes necessary, propose `numeric-core` as a
  deliberate migration; do not fork the core inside another repository.
- Keep the Lua surface **1-based** for dimensions, indices, labels, sort/topk,
  argmax, and gather/scatter. `.npy` file bytes stay NumPy-compatible; docs must
  be explicit where NumPy examples would differ.
- `luatorch` may use ATen for heavy kernels. `numlua` and `luaplotlib` must not
  link libtorch directly unless there is a written design decision.
- Prefer explicit, measurable performance work: benchmark against eager
  PyTorch/NumPy where relevant, use best-of-N on this thermally throttled CPU,
  and report losses as honestly as wins.

### Dependency direction

```
luatorch
  ├── numlua
  └── luaplotlib (through numlua arrays; optional direct tensor adapter)

tensorflow-lua (standalone; TensorFlow C API)
lua-polars     (standalone; Rust Polars C ABI)
luar           (standalone; Lua 5.5 runtime with FFI/JIT)
```

A change that alters a public luatorch API used by `numlua` is a cross-repo
change. Queue it as one task that names both repositories, or land the core
change first with a compatibility shim.

## Cross-repo tasks

The scheduler accepts a multi-repo task header:

```markdown
---
title: Add numlua broadcasting tests
repos: luatorch, numlua
---

Implement ... Acceptance: ...
```

For such a task, the agent gets one worktree per repository on the same branch
slug, runs each repository's tests, and commits per repository. If one repo
fails, do not discard the other worktree. The coordinator reviews the linked
branches and may land them on `main` directly; the scheduler does not auto-merge
or open pull requests unless configured to.

A queue task must be a focused brief with acceptance criteria, not "improve the
project". Empty queues are allowed; the queue scheduler never invents filler
work. The permanent service is the deliberate exception: its workers choose
their own concrete tasks each cycle from the repository roadmap, tests, and
known correctness or performance gaps.

## Tests and toolchain

Run from the repository root unless noted:

```sh
luatorch:              ./scripts/test.sh
numlua:                ./scripts/test.sh
luaplotlib:            ./scripts/test.sh
tensorflow-lua:        ./scripts/test.sh
tensorflow-lua-addons: ./scripts/test.sh
lua-polars:            ./scripts/test.sh
lua-notebooks:         make test
luar:                  make test
numeric-lua-agents:    python3 service/tests/test_scheduler.py
numeric-lua-agents:    bash tests/promo-agents.sh
```

Additional luatorch checks:

```sh
./scripts/compare.sh            # luatorch vs eager PyTorch
./scripts/compare.sh --compile  # also torch.compile baseline; slow
```

Rules:

- Every behavior change needs a test, including an error/fallback path.
- Do not make benchmark timing a hard CI gate. Record numbers and use broad
  thresholds; the CPU throttles by 10–20% between runs.
- Do not download libtorch or rebuild it per agent. Reuse the cached
  `third_party/libtorch` and `build/` directories.
- Keep generated caches (`.luatorch-cache/`, `build/`, `.venv-bench/`,
  `.uv-cache/`, `.uv-python/`) out of commits.
- Do not commit generated plots, binaries, or logs; put them under ignored
  scratch paths.

## Agent services

Two runners exist and share one host-wide lock. Run at most one of them.

### Permanent project service (installed)

`numeric-lua-agents/service/scheduler.py` runs as a user systemd unit. It
resumes one fixed headless DSH session per project, one worker at a time, in a
14-slot rotation where `luar` gets half the slots and each other project gets
one. Weekdays run 07:00–22:00 America/Sao_Paulo; weekends are continuous. After
each worker it reads the account's GitHub contribution count and sleeps until
the next day once `contribution_min` (100) is reached.

- The coordinator owns the unit, the quota, the project list, and the prompts.
- Workers run in the project's main checkout, commit directly to `main`, push
  after the repository's tests pass, and keep the checkout clean.
- Workers do not tag, release, bump versions, or edit rockspecs; the coordinator
  cuts releases from `## Unreleased` entries.
- Do not start a second worker by hand or edit the checkout while a worker is
  running.

### Queue scheduler (manual)

The queue scheduler is adapted from the Maria Padilha scheduler. It runs at
most one headless DSH agent, inside configured discount windows, on
coordinator-written tasks.

- The **coordinator** (the human/DSH main session) owns queues and priorities.
- The scheduler owns **when** and **which repository**; it does not invent work.
- Promo hours are configuration. Verify the provider's current schedule before
  changing windows; the Maria config points at
  <https://api-docs.deepseek.com/news/news260813> as its source.
- Use a stop margin and a `max_tasks_per_day` cap. Defaults keep
  `push = false`, `open_pr = false`, and `maintenance.enabled = false`; the
  coordinator may authorize `push = true` and `push_main = true` per repository.
- The scheduler must never run a task in a dirty checkout, must use worktrees,
  and must finalize work on `SIGTERM` before the window closes.
- A host-wide lock prevents overlapping schedulers. If the Maria scheduler does
  not honor the same lock, run only one of them.

## Code and docs

- Match the existing style; no new runtime dependencies without a written
  reason. LuaJIT is the primary target.
- Public API changes need API docs and a README example.
- Prefer small modules and dependency-free test harnesses.
- Keep licenses BSD-3-Clause unless a dependency forces otherwise; record
  third-party licenses in `third_party/` or a NOTICE.
- Report handoffs with: files changed, exact commands/tests run, benchmark
  numbers (with conditions), known limitations, and anything left uncommitted.
