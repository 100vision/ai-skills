---
name: openvas-cli
description: Manage Greenbone Community Edition (OpenVAS) vulnerability scanners remotely via CLI. Use this skill whenever the user wants to install, configure, or operate openvas-cli for vulnerability scanning, scan management, credential management, target creation, task orchestration, or report retrieval from a Greenbone/OpenVAS instance. Also use when the user mentions OpenVAS, Greenbone CE, gvm-cli, vulnerability assessment, network scanning, CVE scanning, or needs to set up SSH-based remote access to an OpenVAS scanner.
---

# openvas-cli Skill

This skill enables an AI agent to manage Greenbone Community Edition (OpenVAS) instances from a remote workstation using `openvas-cli`, a Python wrapper around `gvm-cli`.

## Architecture

```
#Option 1 : if through a direct ssh connection:
remote workstation(openvas-cli/ai agent)   ──SSH:22──▶  openvas-host  ──▶  gvm-cli socket  ──▶  gvmd.sock

#Option 2 : if OpenVAS is not directly reachable and network path must go through a ssh jump host:
  remote workstation(openvas-cli/ai agent)  ──SSH:22──▶  jump-host  ══tunnel══▶  openvas-host:22
       │                                                 ▲
       └──SSH to localhost:LOCAL_PORT────────────────────┘
           (existing commands see this as a direct connection)
```

The default transport is **SSH**, which tunnels `gvm-cli socket` commands to a remote OpenVAS host. This is specifically designed for Greenbone Community Edition where `gvm-cli ssh` is not available.

## Quick Start

If the user hasn't installed openvas-cli yet, follow this sequence:

1. **Install** — see [Installation](#installation)
2. **Onboard** — `openvas-cli onboard` (interactive setup)
3. **Verify** — `openvas-cli doctor` then `openvas-cli system version`
4. **Use** — discover resources, create scans, retrieve reports

## Installation

Install dependencies on the administrative workstation:

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-pipx python3-venv ssh sshpass
python3 -m pipx install gvm-tools
```

Install openvas-cli:

```bash
git clone https://github.com/100vision/openvas-cli.git
cd openvas-cli
chmod +x ./install.sh
./install.sh install
```

Verify installation:

```bash
./install.sh status
gvm-cli --version
openvas-cli --help
```

**Important**: `openvas-cli` must be installed on the same machine where the agent runs. It also requires a **local** `gvm-cli` installation even when connecting remotely.

## Onboarding

for first-run , you must to run `openvas-cli onboard` to configure the connection to the OpenVAS host you want to manage. 
you could instruct your AI agent to set this up for you or manually do it yourself.


The onboarding basically does these things:

1. Asks for remote OpenVAS host, ssh port, and SSH username and ssh jumphost/bastion information if selected. 
2. Generates a local SSH keypair if missing (`~/.ssh/openvas_cli_ed25519`)
3. Adds the remote host to `~/.ssh/known_hosts`
4. Installs the public key into the remote user's `~/.ssh/authorized_keys`
5. Stores all config in `~/.config/openvas-cli/openvas-cli.conf`

>Note:
>Config file permissions are set to `600`. Re-run with `--force` to rewrite config when settings change.


>Note:
> if using a ssh jumphost/bastion,make sure to use the same ssh user on both jumphost/basion and your OpenVAS host. Otherwise ssh tunnels may not be established.


## Verification Flow

After onboarding, always verify:

```bash
openvas-cli doctor
openvas-cli system version
openvas-cli config list
openvas-cli scanner list
openvas-cli task list
```

If these succeed, the environment is healthy. Proceed with normal operations.

## Command Reference

For detailed command patterns, see `references/commands.md`.

### Discover Resources

```bash
openvas-cli config list 
openvas-cli scanner list
openvas-cli credential list
openvas-cli task list
openvas-cli target list
```

### Inspect Objects

```bash
openvas-cli config get --name "Full and Fast" --details
openvas-cli credential get --name "Windows Admin" --details
openvas-cli task get --name "Weekly Scan"
```

### Create a Scan

```bash
openvas-cli scan create \
  --hosts 192.168.11.10-254 \
  --credential WindowsServer \
  --scan-config "Window-ClientOS" \
  --port-list "All IANA assigned TCP"
```

### Reports

```bash
openvas-cli report list
openvas-cli report get --id REPORT_ID
openvas-cli report get --id REPORT_ID --format pdf --output report.pdf
```

### Credential Management

```bash
openvas-cli credential create --name "Windows Admin" --type up --username administrator
openvas-cli credential create --name "Linux Root" --type usk --username root --private-key ~/.ssh/id_rsa
openvas-cli credential create --name "Router SNMP" --type snmp --community public
```

## Safety Rules

- Always run `openvas-cli onboard` before first use on a new machine
- Prefer saved config over passing secrets on the command line
- Do not delete credentials that may still be attached to targets
- Verify with `openvas-cli doctor` after transport or credential changes
- Before `scan create`, validate: transport is healthy, scan config exists, scanner exists, credential exists (if referenced), valid port list/range

## Troubleshooting

For detailed troubleshooting, see `references/troubleshooting.md`.

### If `doctor` fails, check in order:

1. Remote SSH login works
2. Remote `gvm-cli` exists
3. Remote socket path is correct
4. OpenVVAS services are up and running. if not,run `sudo gvm-start`
5. Remote ssh user has right to access the socket (must be member of `_gvm` group on OpenVAS instance)
6. GMP credentials are correct

### Common Q&A

- **`gvm-cli` exists remotely but not locally?** Install local `gvm-cli` first — openvas-cli depends on the local toolchain.
- **`sshpass` missing?** Install it, then rerun onboarding with `--force`.
- **Remote `gvm-cli` not in PATH?** Set `OPENVAS_REMOTE_GVM_CLI_BIN` to the explicit path.
- **Wrong socket path?** Update `OPENVAS_SOCKET_PATH` in config. Common alternatives: `/run/gvm/gvmd.sock`, `/run/openvas/openvasmd.sock`.
- **Credential deletion fails?** It's likely in use by a target. Remove the association first.
- **SSH works but socket command fails?** Verify the SSH user is a member of `_gvm` group on the remote instance.
