# numeric-lua organization profile

This repository holds the public organization profile in `profile/README.md`,
the organization-wide agent policy in `AGENTS.md`, and the support policy in
`SUPPORT.md`.

The organization is being bootstrapped from local checkouts. See the workspace
`README.md` next to the repositories for the architecture and dependency rules.

Scheduled agents are limited to the library repositories configured in
`~/.config/numeric-lua/agents.toml`. This repository (`.github`) and
`numeric-lua-agents` are coordinator-only: they are deliberately excluded from
agent queues. No agent may merge, release, or change organization settings.
