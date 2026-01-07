# Configure Border0 Network

GitHub Action to configure Border0 Networking on your GitHub Action Runner.

## Usage

You can use this action in two ways:

### Option 1: With Identity Federation (Recommended)

Use GitHub's OIDC identity to obtain Border0 credentials automatically. The action will call `configure-border0-credentials` internally.

```yaml
name: Use Border0 Network with Identity Federation
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Start Border0 VPN
        id: border0-vpn
        uses: borderzero/configure-border0-network@v0.0.1
        with:
          border0-org-subdomain: myorg
          border0-svc-account-name: github-actions
          border0-token-duration-seconds: 900

      - name: Perform Operations Over Border0 Network
        run: |
          # You can now access resources over the Border0 network
          ./do-things-over-border0.sh

      - name: Stop Border0 VPN
        if: always()
        run: bash ${{ steps.border0-vpn.outputs.cleanup-script }}
```

### Option 2: With Direct Token

Use a pre-existing Border0 token:

```yaml
name: Use Border0 Network with Token
on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Start Border0 VPN
        id: border0-vpn
        uses: borderzero/configure-border0-network@v0.0.1
        with:
          border0-token: ${{ secrets.BORDER0_TOKEN }}

      - name: Perform Operations Over Border0 Network
        run: |
          # You can now access resources over the Border0 network
          ./do-things-over-border0.sh

      - name: Stop Border0 VPN
        if: always()
        run: bash ${{ steps.border0-vpn.outputs.cleanup-script }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `border0-token` | Border0 authentication token (if not provided, will require identity federation parameters) | No* | - |
| `border0-org-subdomain` | Border0 organization name (used with identity federation) | No* | - |
| `border0-svc-account-name` | Border0 service account name (used with identity federation) | No* | - |
| `border0-token-duration-seconds` | Token duration in seconds (used with identity federation) | No | `3600` |
| `vpn-wait-seconds` | Seconds to wait for VPN connection to establish | No | `5` |
| `debug` | Enable debug output (shows VPN peers and network diagnostics) | No | `false` |

**Note:** Either `border0-token` OR both `border0-org-subdomain` and `border0-svc-account-name` must be provided.

## Outputs

| Output | Description |
|--------|-------------|
| `cleanup-script` | Path to the cleanup script for stopping the Border0 VPN node (`/tmp/border0-cleanup.sh`) |

## Important Notes

### Cleanup

The Border0 VPN node runs as a background process until manually stopped. **You must add a cleanup step** to stop it, typically with `if: always()`.

#### Recommended: Use the cleanup script output

The action provides a `cleanup-script` output that points to a ready-to-use cleanup script:

```yaml
- name: Start Border0 VPN
  id: border0-vpn
  uses: borderzero/configure-border0-network@v0.0.1
  with:
    border0-org-subdomain: myorg
    border0-svc-account-name: github-actions

- name: Stop Border0 VPN
  if: always()
  run: bash ${{ steps.border0-vpn.outputs.cleanup-script }}
```

#### Alternative: Manual cleanup

You can also manually stop the VPN node:

```yaml
- name: Stop Border0 VPN
  if: always()
  shell: bash
  run: |
    if [ -f /tmp/border0-node.pid ]; then
      PID=$(cat /tmp/border0-node.pid)
      echo "Stopping Border0 VPN node (PID: $PID)..."
      sudo kill $PID 2>/dev/null || true
      sleep 2
      sudo kill -9 $PID 2>/dev/null || true
      rm -f /tmp/border0-node.pid
      echo "Border0 VPN node stopped"
    fi
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
