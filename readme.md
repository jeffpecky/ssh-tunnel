## GitHub Action SSH Tunnel via Tailscale

A GitHub Action for connecting to the runner via SSH over your [Tailscale](https://tailscale.com/) tailnet.

### Why?

Debugging GitHub Actions remotely can be difficult. This action connects the runner to your Tailnet so you can SSH directly over the Tailscale network — no public relay needed.

### Requirements

1. A [Tailscale](https://tailscale.com/) account
2. An [OAuth client](https://tailscale.com/docs/features/oauth-clients) with the `auth_keys` (Write) scope and a tag (e.g. `tag:ci`)
3. An SSH public key (e.g. `~/.ssh/id_rsa.pub`)
4. A runner image version >= 2.237.1

### Setup

Create a YAML workflow (e.g. `ssh.yml`) in `.github/workflows` following this example:

```yaml
name: SSH Tunnel
on: push

jobs:
  deploy:
    name: Set up tunnel
    runs-on: ubuntu-24.04
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup tunnel
      uses: jeffpecky/ssh-tunnel@main
      with:
        timeout: 6h
        ssh_public_key: ${{ secrets.SSH_PUBLIC_KEY }}
        ts_oauth_client_id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
        ts_oauth_secret: ${{ secrets.TS_OAUTH_SECRET }}
```

### Required Secrets

Create the following repository secrets (Settings -> Secrets -> New repository secret):

`SSH_PUBLIC_KEY`: your local SSH public key (e.g. `~/.ssh/id_rsa.pub`)

`TS_OAUTH_CLIENT_ID`: your Tailscale OAuth client ID

`TS_OAUTH_SECRET`: your Tailscale OAuth client secret

### Outputs

| Output | Description |
|--------|-------------|
| `tailscale-ip` | Tailscale IPv4 address of the runner node |

### Deploy

On the next push, the action will:
1. Register the runner as an ephemeral Tailscale node
2. Start the SSH server
3. Display the Tailscale IP for SSH access
4. Keep the runner alive for the configured timeout

### Connect via SSH

The runner username is `runner`. Use the Tailscale IP printed in the action logs:

```
$ ssh runner@100.x.x.x
```
