# Google Colab CLI SSH Reference

Primary source: https://github.com/googlecolab/google-colab-cli/blob/main/docs/06_ssh_access.md. That document and `colab ssh --help` are version-sensitive; confirm current behavior before acting.

## What the connection is

`colab ssh` is an SSH client workflow that bridges Colab's runtime SSH endpoint through a WebSocket and the Colab session credentials. It is not a direct connection to a stable VM IP or a user-managed public port. Current upstream docs state the runtime must expose the `/colab/ssh` endpoint; older or unsupported runtime sessions can return 404. A new session may be needed if the feature endpoint is not available.

The CLI currently supports Linux and macOS. Check `colab version` and `colab ssh --help` first.

## Keys

Current upstream SSH docs accept `ssh-ed25519` and ECDSA keys `ecdsa-sha2-nistp256`, `nistp384`, or `nistp521`. RSA keys (`ssh-rsa`) are rejected. By default, the CLI looks for `~/.ssh/id_ed25519` and `~/.ssh/id_ecdsa`; `-i`/`--identity` selects a private key explicitly.

Check available public key types without exposing private key content:

```bash
ls -l ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

If a key must be created, preserve existing keys and ask before changing local credentials. Example only, after authorization and confirming the path does not exist:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/colab_ed25519 -C colab-cli
```

Never copy or print private key data. The CLI passes the public key to the Colab SSH endpoint; the private key remains local for the local SSH client.

## Interactive shell

```bash
colab ssh
colab ssh -s my-session
colab ssh -s my-session -i ~/.ssh/colab_ed25519
```

Current docs: if no sessions exist, bare SSH can auto-create a runtime; if one exists, it can select it; if multiple exist, specify `-s`. To avoid ambiguity, use `-s` explicitly. `--gpu`, `--tpu`, and `--high-mem` apply to a runtime created by this command, not an already-existing session. Current docs say interactive shell starts in `/content` and forces a PTY.

`--rm` is lifecycle-sensitive. For interactive use, current upstream behavior documents stopping a runtime auto-created by `colab ssh` at exit; a reused session is not removed. Check help before relying on it.

## OpenSSH ProxyCommand

`--proxy-mode` bridges standard input/output to the Colab SSH WebSocket and is intended for OpenSSH clients and SSH-based IDEs. Example `~/.ssh/config` entry from upstream, adapted to use a named alias:

```sshconfig
Host colab-work
  HostName colab-work
  User root
  ProxyCommand /absolute/path/to/colab ssh --proxy-mode -s work
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  RequestTTY yes
  RemoteCommand cd /content 2>/dev/null; exec bash -l
```

Find the executable path with `command -v colab`, then use that absolute path in `ProxyCommand`; SSH may invoke it in a non-login shell without the usual PATH. For one-off CLI testing, OpenSSH config is unnecessary:

```bash
ssh -o 'ProxyCommand=/absolute/path/to/colab ssh --proxy-mode -s work' root@colab-work
```

The exact target/argument form can vary with local OpenSSH or IDE behavior; test the alias with a simple SSH command first. The upstream CLI example includes `User root`, `StrictHostKeyChecking no`, and `UserKnownHostsFile /dev/null` because this is a dynamic proxy endpoint. Scope these options to the Colab alias only; do not disable host checking globally. Evaluate security requirements before adopting these settings.

`--proxy-mode` supports session selection and, per current docs, can create a missing named session. Accelerator flags can be applied during that creation. `--rm` can request teardown on bridge disconnect. Be careful using it with IDEs that disconnect/reconnect automatically; they may stop the runtime on a transient disconnect.

## Troubleshooting

- **No identity or unsupported key (400):** Select or create an Ed25519/ECDSA key; do not use RSA.
- **401:** Session token may be invalid or expired. Reauthenticate, inspect active sessions, and create a fresh session only if needed.
- **403:** Permission or feature access is denied. Check account entitlement and applicable Colab policy; do not attempt to bypass it.
- **404:** Runtime may not expose the SSH endpoint. Use a supported current runtime/session; avoid endless retries.
- **429:** Another SSH connection may already be active. Close the existing connection first.
- **502:** Runtime SSH daemon may be unavailable or unhealthy. Check `colab status` and reconnect once; stop/recreate only if appropriate.
- **Several sessions:** Specify `-s NAME`.
- **ProxyCommand cannot find `colab`:** Use the absolute executable path from `command -v colab`.
- **Proxy/IDE protocol breaks:** Keep stdout exclusively as the SSH byte stream. Do not add shell wrappers or startup messages that print to stdout. Current CLI routes its own proxy-mode status output to stderr.
- **Connection drops/disconnect cleanup:** Check whether `--rm` was used and whether the IDE reconnects. Check `colab sessions` and stop only the intended leftover session.

## Policy

Google's Colab FAQ says resources are not guaranteed and usage is limited. It identifies remote control such as SSH shells or remote desktops and bypassing notebook UI for primary interaction as potentially disallowed on free usage in specified circumstances without a positive compute-unit balance. Other restricted activities include service hosting, proxying, cryptocurrency mining, and circumventing anti-abuse limits. Check current FAQ and terms before SSH use: https://research.google.com/colaboratory/faq.html

Do not treat the presence of a working CLI command as authorization for every use case. For persistent remote development, service hosting, or guaranteed SSH availability, use an environment designed and authorized for that purpose.

## Sources

- CLI SSH design and behavior: https://github.com/googlecolab/google-colab-cli/blob/main/docs/06_ssh_access.md
- CLI README: https://github.com/googlecolab/google-colab-cli
- Google Colab FAQ: https://research.google.com/colaboratory/faq.html
