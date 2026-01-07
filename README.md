# Configure Border0 Network

GitHub Action to install the Border0 CLI and start a VPN node.

## Features

- Automatically detects Linux architecture (amd64, arm64, arm)
- Downloads and installs the appropriate Border0 CLI binary
- Starts a Border0 VPN node in ephemeral mode
- Provides cleanup mechanism to stop the node

## Usage

### Basic Example

```yaml
name: Use Border0 Network
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Start Border0 VPN
        uses: ./configure-border0-network
        with:
          border0-token: ${{ secrets.BORDER0_TOKEN }}

      - name: Run tests with Border0 network access
        run: |
          # Your tests that need Border0 network access
          echo "Running tests..."

      - name: Stop Border0 VPN
        if: always()
        run: |
          bash ${{ github.action_path }}/cleanup.sh
```

### With Other Border0 Actions

```yaml
name: Use Border0 with OIDC
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Get Border0 credentials
        id: border0-creds
        uses: ./configure-border0-credentials
        with:
          border0-org-subdomain: myorg
          border0-svc-account-name: github-actions

      - name: Start Border0 VPN
        uses: ./configure-border0-network
        with:
          border0-token: ${{ steps.border0-creds.outputs.token }}

      - name: Run tests
        run: |
          # Tests with network access
          npm test

      - name: Stop Border0 VPN
        if: always()
        run: |
          bash ./configure-border0-network/cleanup.sh
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `border0-token` | Border0 authentication token | Yes | - |

## Important Notes

### Cleanup

The Border0 VPN node runs as a background process until manually stopped. **You must call the cleanup script** to stop it, typically in a cleanup step with `if: always()`:

```yaml
- name: Stop Border0 VPN
  if: always()
  run: bash ./configure-border0-network/cleanup.sh
```

### Log File

The Border0 node output is logged to `/tmp/border0-node.log` for debugging purposes.

### Supported Architectures

- Linux x86_64 (amd64)
- Linux aarch64/arm64
- Linux armv7l/arm

## Troubleshooting

### Check if the node is running

```yaml
- name: Check Border0 status
  run: |
    ps aux | grep border0
    tail -n 20 /tmp/border0-node.log
```

### Node fails to start

Check the log file for errors:

```yaml
- name: Debug Border0
  if: failure()
  run: cat /tmp/border0-node.log
```

## License

Same as the repository license.
