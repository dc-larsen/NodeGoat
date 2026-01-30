# Socket Security Tier 1 Reachability Analysis Setup

This guide explains how to set up Socket Security's Tier 1 reachability analysis using GitHub Actions. Reachability analysis identifies which vulnerabilities in your dependencies are actually reachable from your application code, reducing alert noise and helping you prioritize fixes.

## Prerequisites

- Socket Security enterprise plan
- GitHub repository with supported package ecosystems (npm, PyPI, Maven, Go, etc.)
- Socket API token with the following scopes:
  - `socket-basics` (or `socket-basics:read`)
  - `uploaded-artifacts` (or `uploaded-artifacts:create`)
  - `full-scans` (or `full-scans:create`, `full-scans:list`)
  - `repo` (or `repo:create`)

## Setup Instructions

### 1. Create a Socket API Token

1. Go to [Socket Dashboard](https://socket.dev/dashboard) and navigate to your organization settings
2. Create a new API token with the required scopes listed above
3. Copy the token (you won't be able to see it again)

### 2. Add the API Token to GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** > **Secrets and variables** > **Actions**
3. Click **New repository secret**
4. Name: `SOCKET_SECURITY_API_KEY`
5. Value: Paste your Socket API token
6. Click **Add secret**

### 3. Add the Workflow File

Create a new file at `.github/workflows/socket-security-scan.yml` with the following content:

```yaml
# Socket Security Scan with Tier 1 Reachability Analysis
#
# This workflow scans your dependencies and performs reachability analysis
# to identify which vulnerabilities are actually reachable in your code.
#
# Required: SOCKET_SECURITY_API_KEY secret with enterprise plan
# API token scopes needed: socket-basics, uploaded-artifacts, full-scans, repo

name: Socket Security Scan

on:
  schedule:
    - cron: "0 6 * * 1"  # Weekly on Monday at 6 AM UTC
  workflow_dispatch:
    inputs:
      enable_reachability:
        description: 'Enable Tier 1 reachability analysis'
        required: false
        default: 'true'
        type: choice
        options:
          - 'true'
          - 'false'

concurrency:
  group: socket-security-scan
  cancel-in-progress: true

jobs:
  socket-security:
    name: Socket Security Scan
    runs-on: ubuntu-latest
    timeout-minutes: 60
    permissions:
      contents: read

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install uv (Python package manager)
        uses: astral-sh/setup-uv@v4

      - name: Install Socket CLI
        run: pip install socketsecurity --upgrade

      - name: Run Socket Security Scan
        env:
          SOCKET_SECURITY_API_KEY: ${{ secrets.SOCKET_SECURITY_API_KEY }}
          PYTHONUNBUFFERED: "1"
        run: |
          REPO_NAME="${GITHUB_REPOSITORY#*/}"

          # Build reachability flags if enabled
          REACH_FLAGS=""
          if [[ "${{ github.event.inputs.enable_reachability }}" != "false" ]]; then
            REACH_FLAGS="--reach --reach-memory-limit 16384 --reach-timeout 3600"
            echo "Reachability analysis enabled"
          fi

          echo "Scanning repository: $REPO_NAME"

          socketcli \
            --target-path "$GITHUB_WORKSPACE" \
            --repo "$REPO_NAME" \
            --enable-debug \
            $REACH_FLAGS
```

### 4. Trigger the Workflow

You can trigger the workflow in two ways:

**Manual trigger:**
1. Go to your repository's **Actions** tab
2. Select "Socket Security Scan" from the workflows list
3. Click **Run workflow**
4. Choose whether to enable reachability analysis
5. Click **Run workflow**

**Automatic schedule:**
The workflow runs automatically every Monday at 6 AM UTC.

## Configuration Options

### Reachability Analysis Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--reach` | - | Enable reachability analysis |
| `--reach-memory-limit` | 8192 | Memory limit in MB for analysis |
| `--reach-timeout` | 1800 | Timeout in seconds for analysis |

### Customizing the Schedule

Modify the cron expression to change when the scan runs:

```yaml
schedule:
  - cron: "0 6 * * 1"  # Every Monday at 6 AM UTC
  - cron: "0 6 * * *"  # Every day at 6 AM UTC
  - cron: "0 */6 * * *"  # Every 6 hours
```

## Supported Ecosystems

Reachability analysis supports:
- **npm** (JavaScript/TypeScript)
- **PyPI** (Python)
- **Maven** (Java)
- **Go modules**

## Viewing Results

After the scan completes:

1. Go to the [Socket Dashboard](https://socket.dev/dashboard)
2. Navigate to your organization
3. Select the repository
4. View the scan results with reachability data

Vulnerabilities will show:
- **Reachable**: The vulnerable code path is reachable from your application
- **Not Reachable**: The vulnerable code is not called by your application
- **Unknown**: Reachability could not be determined

## Troubleshooting

### Common Issues

**"Multiple repositories found with the same slug"**
- Use a unique `--repo` name in the workflow
- Example: `--repo "MyRepo-unique-suffix"`

**"API token unauthorized"**
- Verify the token has the required scopes
- Check that `SOCKET_SECURITY_API_KEY` secret is set correctly

**Reachability analysis timeout**
- Increase `--reach-timeout` value
- Increase `--reach-memory-limit` for large codebases

**"Organization plan does not support reachability"**
- Reachability analysis requires an enterprise plan
- Contact Socket support to upgrade

### Debug Mode

The workflow includes `--enable-debug` by default. Check the GitHub Actions logs for detailed output.

## Example Output

A successful reachability scan will show:

```
Reachability analysis enabled
Starting reachability analysis...
Found 5 manifest files for reachability upload
Running reachability analysis for 145 vulnerabilities
Reachability analysis completed successfully
Results written to: .socket.facts.json
```

## Support

- [Socket Documentation](https://docs.socket.dev)
- [Socket Support](mailto:support@socket.dev)
- [GitHub Issues](https://github.com/SocketDev/socket-python-cli/issues)
