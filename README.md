# Socket Tier 1 Reachability Example

Example repository demonstrating Socket's Tier 1 reachability analysis using the Socket CLI in GitHub Actions.

## What is Reachability Analysis?

Reachability analysis determines which vulnerabilities in your dependencies are actually callable from your application code. This reduces alert noise by distinguishing between theoretical vulnerabilities and those that pose real risk.

## About This Example

This repository runs Socket's reachability analysis on [OWASP NodeGoat](https://github.com/OWASP/NodeGoat), a deliberately vulnerable Node.js application. View the [workflow runs](../../actions) to see reachability in action.

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
  # Run weekly on Monday at 6 AM UTC
  schedule:
    - cron: "0 6 * * 1"
  # Allow manual triggers from GitHub Actions UI
  workflow_dispatch:

jobs:
  socket-security:
    runs-on: ubuntu-latest
    timeout-minutes: 60
    permissions:
      contents: read

    steps:
      # Full checkout required for reachability to analyze code paths
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Python 3.12+ required for Socket CLI
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      # Node.js 20+ required for reachability analysis engine
      - uses: actions/setup-node@v4
        with:
          node-version: "20"

      # uv required for Python dependency resolution in reachability
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

          # --reach                   Enable Tier 1 reachability analysis
          # --reach-memory-limit      Memory limit in MB (default: 8192)
          # --reach-timeout           Timeout in seconds (default: 1800)
```

## Supported Ecosystems

npm, PyPI, Maven, Go modules

## Resources

- [Socket Dashboard](https://socket.dev/dashboard)
- [Socket Documentation](https://docs.socket.dev)
- [Socket CLI](https://github.com/SocketDev/socket-python-cli)
