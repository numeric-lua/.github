# numeric-lua organization profile

This repository holds the public organization profile in `profile/README.md`,
the organization-wide agent policy in `AGENTS.md`, and the support policy in
`SUPPORT.md`.

The organization is being bootstrapped from local checkouts. See the workspace
`README.md` next to the repositories for the architecture and dependency rules.

The permanent project service rotates the library repositories listed in
`numeric-lua-agents/service/scheduler.py`; the queue scheduler is limited to the
repositories configured in `~/.config/numeric-lua/agents.toml`. This repository
(`.github`) and `numeric-lua-agents` are coordinator-only and are deliberately
excluded from both. No agent may release, tag, bump a version, or change
organization settings.
