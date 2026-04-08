# openvas-cli Troubleshooting Guide

## If `doctor` Fails

Check in this exact order:

1. **Remote SSH login works**
   ```bash
   ssh user@host
   ```

2. **Remote `gvm-cli` exists**
   ```bash
   ssh user@host 'command -v gvm-cli'
   ```

3. **Remote socket path is correct**
   ```bash
   ssh user@host 'ls -l /run/gvmd/gvmd.sock'
   ```
   Common socket paths:
   - `/run/gvmd/gvmd.sock` (default)
   - `/run/gvm/gvmd.sock`
   - `/run/openvas/openvasmd.sock`
   - `/usr/share/gvm/gsad/web/gvmd.sock`
   - `/usr/share/openvas/gsa/classic/openvasmd.sock`

4. **Remote user can access the socket**
   ```bash
   ssh user@host 'id ssh_user_name'
   ```
   The SSH user must be a member of the `_gvm` group.

5. **GMP credentials are correct**
   ```bash
   ssh user@host 'gvm-cli --gmp-username admin --gmp-password pass socket --socketpath /run/gvmd/gvmd.sock --xml "<get_version/>"'
   ```

## If Greenbone CE Remote Access Is Broken

**Do not** assume `gvm-cli ssh` is supported — it typically isn't on Community Edition.

Prefer:
- `openvas-cli` SSH wrapper mode (default)
- Direct remote socket execution over plain SSH

## Q&A

### Q: What if `gvm-cli` exists on the remote host but not on the local host?

`openvas-cli` requires a **local** `gvm-cli` installation because it depends on the local toolchain and command model. Install local `gvm-cli` first:

```bash
python3 -m pipx install gvm-tools
```

Then continue with onboarding.

### Q: What if `sshpass` is missing during SSH onboarding?

SSH onboarding bootstrap needs `sshpass` one time to install the generated public key using the provided SSH password.

```bash
sudo apt-get install -y sshpass
```

Then rerun onboarding with `--force` flag.

### Q: What if SSH login works but the public key cannot be installed remotely?

Check whether the remote user can create or update files:

```bash
ssh user@host 'ls -la ~/.ssh/'
ssh user@host 'ls -la ~/.ssh/authorized_keys'
```

If not, fix remote home directory or permission issues first.

### Q: What if the generated SSH key exists locally but remote key install never happened?

Treat onboarding as incomplete. Re-run:

```bash
openvas-cli onboard --force
```

Allow it to reinstall the public key, or install the public key manually on the remote host.

### Q: What if the remote `gvm-cli` is not in `PATH`?

Set or save the explicit remote path in config:

```bash
OPENVAS_REMOTE_GVM_CLI_BIN="/usr/local/bin/gvm-cli"
```

### Q: What if the remote socket is not `/run/gvmd/gvmd.sock`?

Use the actual socket path in config:

```bash
OPENVAS_SOCKET_PATH="/run/gvm/gvmd.sock"
```

Always verify with a remote socket test if SSH mode fails.

### Q: What if `openvas-cli doctor` fails in SSH mode but the remote socket command works manually?

Check these next:

1. Remote `gvm-cli` path mismatch — verify `OPENVAS_REMOTE_GVM_CLI_BIN` in config
2. Wrong socket path saved in config — verify `OPENVAS_SOCKET_PATH`
3. Wrong SSH identity file — verify `OPENVAS_SSH_IDENTITY_FILE`
4. Changed remote host key in `known_hosts` — remove stale entry
5. Wrong GMP username or password — verify `OPENVAS_GMP_USERNAME` and `OPENVAS_GMP_PASSWORD`

### Q: Should I use `socket` or `ssh` when both are possible?

- Prefer `socket` if the CLI runs on the **same host** as `gvmd`
- Prefer `ssh` for **remote** Greenbone Community Edition access

### Q: When should I stop trying SSH and switch strategy?

If plain SSH works but the remote socket command cannot be made to work reliably:

1. Verify Unix socket access — specifically check if SSH user is a member of `_gvm` group
2. Consider `socket` if running locally on the remote OpenVAS instance
3. Ask OpenVAS administrator to check if OpenVAS is alive and operational
4. Consider `tls` only if GMP over TLS is actually configured

### Q: What should I validate before using `scan create`?

At minimum validate:

1. Transport is healthy via `openvas-cli doctor`
2. The requested scan config exists (`openvas-cli config list`)
3. The scanner exists (`openvas-cli scanner list`)
4. The credential exists if one is referenced (`openvas-cli credential list`)
5. A valid port list or port range is provided

### Q: What should I do if credential deletion fails because the credential is in use?

Do not force deletion by default. First identify which targets reference the credential, remove the association, then retry deletion.

### Q: What minimum checks are required before saying the environment is healthy?

```bash
openvas-cli doctor
openvas-cli system version
openvas-cli config list
openvas-cli scanner list
```

### Q: Does onboarding happen on the local machine or remote OpenVAS instance?

**Both:**
- **Local machine**: generate SSH keypair, save openvas-cli config, update `known_hosts`
- **Remote OpenVAS instance**: install the generated public key into `authorized_keys`

### Q: Does the guide distinguish SSH authentication from GMP authentication?

Yes. SSH authentication is for **reaching** the remote OpenVAS instance. GMP authentication is for **talking to `gvmd`** after the SSH transport is established.

### Q: Is openvas-cli onboarding safe to re-run?

Yes, but it updates local config and may reinstall or refresh SSH bootstrap state. Re-run onboarding with `--force` option to re-write the config when:
- Transport settings change
- Hostnames change
- Credentials change
- Remote paths change
