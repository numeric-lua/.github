# Security policy

Do not open a public issue containing a credential, private path, screenshot, or
exploit. Once the organization is public, use GitHub security advisories for the
affected repository. Before then, contact the coordinator directly.

Secrets:

- `DEEPSEEK_API_KEY` and DSH credentials belong in `~/.dsh` or a mode-600 env
  file outside the repositories.
- Never put credentials in a task, prompt, commit, test fixture, log, or
  screenshot.
- The scheduler and agents must not print environment variables that may contain
  secrets.
