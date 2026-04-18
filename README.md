# vetter

Public integration surface for [vetter](https://vetter.dev) — a closed-loop
PR validator that spins up your preview environment, drives it with a Claude
validation agent, and returns a pass/fail verdict on the PR.

This repo is Apache-2.0 and holds the customer-facing artifacts: the GitHub
Action, the attestation verifier CLI, runnable examples, and integration
docs. The cloud service (orchestration, signing, attestation store) lives
in a separate, closed repo.

## Contents

| Path | What it is |
|---|---|
| [`action/`](./action) | Composite GitHub Action — the supported way to wire vetter into your CI. |
| [`examples/github-actions/`](./examples/github-actions) | Raw workflow YAML if you'd rather copy-paste and customize. |
| [`verify/`](./verify) | `vetter-verify` CLI for offline attestation verification (Phase 2). |

## Quick start

Add a workflow to your repo at `.github/workflows/vetter.yml`:

```yaml
name: vetter

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  validate:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # ... your app's bring-up steps (install, migrate, start on $APP_PORT) ...

      - uses: nithinkrishnamurthi/vetter/action@v1
        with:
          api-key: ${{ secrets.VETTER_API_KEY }}
          env-id: ${{ secrets.VETTER_ENV_ID }}
          daemon-token: ${{ secrets.VETTER_DAEMON_TOKEN }}
          api-base: ${{ vars.VETTER_API_BASE }}
          app-url: http://localhost:3000
```

Drop a `.vetter/ticket.md` at the repo root describing what the PR should
accomplish. vetter reads it, validates the preview env against it, and
comments the verdict on the PR.

For a fully worked example including Postgres bring-up and Prisma migrate,
see [`examples/github-actions/vetter.yml`](./examples/github-actions/vetter.yml).

## What vetter does

1. The action starts a local daemon on the CI runner and opens a
   cloudflared tunnel so the vetter cloud can reach it.
2. The cloud spawns a Claude validation agent that drives the tunnel like
   a browser: navigates the preview app, runs scenarios, checks against the
   ticket's acceptance criteria.
3. The agent returns a structured verdict; the action posts it as a PR
   comment and surfaces `verdict-status` as an output for downstream gating.

## License

Apache-2.0. See [LICENSE](./LICENSE).
