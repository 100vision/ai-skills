# openvas-cli Command Reference

## Config File

Location: `~/.config/openvas-cli/openvas-cli.conf` (permissions `600`)

### SSH Transport Config

```bash
OPENVAS_TRANSPORT="ssh"
OPENVAS_HOST="openvas.example.com"
OPENVAS_PORT="22"
OPENVAS_SSH_USERNAME="ssh-user-name"
OPENVAS_SSH_IDENTITY_FILE="/home/user/.ssh/openvas_cli_ed25519"
OPENVAS_REMOTE_GVM_CLI_BIN="gvm-cli"
OPENVAS_SOCKET_PATH="/run/gvmd/gvmd.sock"
OPENVAS_GMP_USERNAME="admin"
OPENVAS_GMP_PASSWORD="..."
```

### TLS Transport Config (optional fields)

```bash
OPENVAS_TLS_CERTFILE="/path/to/cert.pem"
OPENVAS_TLS_KEYFILE="/path/to/key.pem"
OPENVAS_TLS_CAFILE="/path/to/ca.pem"
```

## Subcommand Reference

All subcommands inherit global options. Use `--json` or `--compact-json` for machine-readable output.

### Global Options

```
--gvm-cli-bin       Path to gvm-cli binary
--config            Path to config file
--env-file          Path to environment file
--timeout           Command timeout in seconds
--gmp-username      GMP username
--gmp-password      GMP password
--transport         Transport: socket, tls, ssh (default: ssh)
--socketpath        Unix socket path
--hostname          Remote hostname
--port              Remote port
--ssh-username      SSH username
--ssh-identity-file SSH identity file path
--auto-accept-host  Auto-accept SSH host keys
--certfile          TLS certificate file
--keyfile           TLS key file
--cafile            TLS CA file
--no-credentials    Skip loading credentials from config
--debug             Enable debug output
--json              JSON output
--compact-json      Compact JSON output
```

### onboard

Interactive setup wizard.

```bash
openvas-cli onboard              # Interactive setup
openvas-cli onboard --force      # Rewrite existing config
openvas-cli onboard --test       # Test mode (no changes)
openvas-cli onboard --path FILE  # Custom config path
```

### doctor

Diagnostic check. Verifies:
- `gvm-cli` is installed locally
- Transport is configured
- Config file exists
- Can reach remote and get version

```bash
openvas-cli doctor
```

### system

```bash
openvas-cli system version       # Get OpenVAS/GMP version
```

### target

```bash
openvas-cli target list [--filter FILTER]
openvas-cli target get --name NAME [--details] [--tasks]
openvas-cli target get --id ID [--details] [--tasks]
openvas-cli target create --name NAME --hosts HOSTS \
  [--exclude-hosts HOSTS] [--credential CRED] \
  [--port-list PORT_LIST] [--port-range RANGE]
openvas-cli target update --name NAME --set-name NEW_NAME
```

### task

```bash
openvas-cli task list [--filter FILTER]
openvas-cli task get --name NAME [--details]
openvas-cli task get --id ID [--details]
openvas-cli task create --name NAME --target TARGET \
  --scan-config CONFIG --scanner SCANNER
openvas-cli task update --name NAME --set-name NEW_NAME
openvas-cli task start --name NAME
openvas-cli task start --id ID
openvas-cli task stop --name NAME
openvas-cli task stop --id ID
openvas-cli task resume --name NAME
openvas-cli task resume --id ID
```

### report

```bash
openvas-cli report list [--filter FILTER]
openvas-cli report get --id REPORT_ID
openvas-cli report get --id REPORT_ID --format xml
openvas-cli report get --id REPORT_ID --format pdf --output report.pdf
```

### config

```bash
openvas-cli config list [--filter FILTER]
openvas-cli config get --name NAME [--details] [--tasks] [--preferences]
openvas-cli config get --id ID [--details] [--tasks] [--preferences]
```

### scanner

```bash
openvas-cli scanner list [--filter FILTER]
```

### credential

```bash
openvas-cli credential list [--filter FILTER]
openvas-cli credential get --name NAME [--details]
openvas-cli credential get --id ID [--details]

# Username + Password
openvas-cli credential create --name "Windows Admin" --type up --username administrator
# Username + SSH Key
openvas-cli credential create --name "Linux Root" --type usk --username root --private-key ~/.ssh/id_rsa
# SNMP
openvas-cli credential create --name "Router SNMP" --type snmp --community public

openvas-cli credential update --name NAME --username NEW_USER
openvas-cli credential delete --name NAME [--force]
```

### report-format

```bash
openvas-cli report-format list [--filter FILTER]
```

### scan (high-level workflow)

The `scan create` command orchestrates: target find/create → task find/create → binding reconciliation → task start.

```bash
openvas-cli scan create \
  --hosts 192.168.11.10-254 \
  --credential WindowsServer \
  --scan-config "Window-ClientOS" \
  --port-list "All IANA assigned TCP" \
  [--target-name CUSTOM_TARGET] \
  [--task-name CUSTOM_TASK] \
  [--scanner SCANNER_NAME]
```

### Pre-scan Validation Checklist

Before running `scan create`:

1. `openvas-cli doctor` — transport is healthy
2. `openvas-cli config list` — requested scan config exists
3. `openvas-cli scanner list` — scanner exists
4. `openvas-cli credential list` — credential exists (if referenced)
5. Valid port list or port range provided

## Manual SSH Verification Commands

When troubleshooting, these direct SSH commands help diagnose issues:

```bash
# Check gvm-cli exists remotely
ssh user@host 'command -v gvm-cli'

# Check socket path
ssh user@host 'ls -l /run/gvmd/gvmd.sock'

# Test GMP connection
ssh user@host 'gvm-cli --gmp-username admin --gmp-password pass socket --socketpath /run/gvmd/gvmd.sock --xml "<get_version/>"'

# Check group membership
ssh user@host 'id ssh_user_name'
```
