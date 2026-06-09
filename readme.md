## GitHub Action SSH Tunnel via OutRay

A GitHub Action for connecting to the runner via SSH using [OutRay](https://outray.dev/) TCP tunnels.

### Why?

Debugging GitHub Actions remotely can be difficult. Maybe you want to connect to the runner environment live to troubleshoot.

### Requirements

1. An [OutRay](https://outray.dev/) account and API key
2. An SSH public key (e.g. `~/.ssh/id_rsa.pub`)

### Compatibility

This Action was only tested on the **Ubuntu 24.04** runner, but it may work on other Linux based runners.

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
        timeout: 1h
        ssh_public_key: ${{ secrets.SSH_PUBLIC_KEY }}
        outray_api_key: ${{ secrets.OUTRAY_API_KEY }}
```

### Required Secrets

Create two repository secrets (Settings -> Secrets -> New repository secret)

`SSH_PUBLIC_KEY`: your local SSH public key (e.g. `~/.ssh/id_rsa.pub`)

`OUTRAY_API_KEY`: your OutRay API key from [outray.dev/settings](https://outray.dev/settings)

### Outputs

| Output | Description |
|--------|-------------|
| `public-url` | OutRay TCP tunnel endpoint (e.g. `tcp://o1.outray.app:20123`) |

### Deploy

On the next push, the action will:
1. Start the SSH server on the runner
2. Install OutRay and create a TCP tunnel exposing port 22
3. Extract and display the tunnel URL
4. Block and keep the runner alive for the configured timeout duration

### Connect via SSH

The runner username is `runner`. Use the tunnel endpoint printed in the action logs:

```
$ ssh -p 20123 runner@o1.outray.app
```
