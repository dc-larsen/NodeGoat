# Socket Tier 1 Reachability Example

Example repository demonstrating Socket's Tier 1 reachability analysis using the Socket CLI in GitHub Actions.

## What is Reachability Analysis?

Reachability analysis determines which vulnerabilities in your dependencies are actually callable from your application code. This reduces alert noise by distinguishing between theoretical vulnerabilities and those that pose real risk.

## Live Example

This repository runs Socket's reachability analysis on [OWASP NodeGoat](https://github.com/OWASP/NodeGoat), a deliberately vulnerable Node.js application.

- [View latest scan results](https://socket.dev/dashboard/org/david-s-github/repo/NodeGoat)
- [View workflow runs](https://github.com/dc-larsen/socket-tier1-reachability-example/actions)

## Quick Start

See the [setup guide](docs/SOCKET_REACHABILITY_SETUP.md) for complete instructions.

### Requirements

- Socket enterprise plan
- API token with scopes: `socket-basics`, `uploaded-artifacts`, `full-scans`, `repo`

### Workflow

Add to `.github/workflows/socket-security-scan.yml`:

```yaml
name: Socket Security Scan

on:
  schedule:
    - cron: "0 6 * * 1"
  workflow_dispatch:

jobs:
  socket-security:
    runs-on: ubuntu-latest
    timeout-minutes: 60
    permissions:
      contents: read

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - uses: actions/setup-node@v4
        with:
          node-version: "20"

      - uses: astral-sh/setup-uv@v4

      - name: Install Socket CLI
        run: pip install socketsecurity --upgrade

      - name: Run Socket Security Scan
        env:
          SOCKET_SECURITY_API_KEY: ${{ secrets.SOCKET_SECURITY_API_KEY }}
        run: |
          socketcli \
            --target-path "$GITHUB_WORKSPACE" \
            --repo "${GITHUB_REPOSITORY#*/}" \
            --reach \
            --reach-memory-limit 16384 \
            --reach-timeout 3600
```

## Supported Ecosystems

npm, PyPI, Maven, Go modules

## Resources

- [Socket Dashboard](https://socket.dev/dashboard)
- [Socket Documentation](https://docs.socket.dev)
- [Socket CLI](https://github.com/SocketDev/socket-python-cli)
