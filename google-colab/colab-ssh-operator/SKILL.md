---
name: colab-ssh-operator
description: Set up and use the official Google Colab CLI's SSH-over-WebSocket connection. Use when an agent needs an interactive shell on a Colab runtime, an OpenSSH ProxyCommand bridge, or IDE remote development through Colab SSH.
---

# Google Colab SSH Operator

Use the official `colab ssh` command for SSH access to a Colab runtime. This is Colab CLI's WebSocket-mediated SSH feature, not a conventional public-IP/port-22 connection. Read [the bundled SSH reference](references/ssh-reference.md) for exact options and OpenSSH configuration examples. Check the installed CLI's `colab ssh --help` because command behavior and runtime support can vary by release.

## Workflow

1. **Check prerequisites.** Confirm `colab` is installed and current, the machine is Linux/macOS, and an acceptable SSH identity is available. Current upstream docs accept Ed25519 or ECDSA keys and reject RSA. Prefer an existing Ed25519 key. If none exists, create one only with user authorization and never overwrite an existing key.
2. **Check policy suitability first.** SSH shell/remote control may be restricted on free Colab usage. Review Google's current Colab FAQ/terms and the user's plan/workload. Do not use SSH to bypass notebook policies or keep prohibited services alive. If unsuitable, explain the constraint and suggest an authorized environment.
3. **Select the runtime intentionally.** Use `-s SESSION` to avoid ambiguity. Current upstream docs describe auto-creating a runtime if none exists; multiple active sessions require an explicit choice. Pass accelerator flags only when needed and only when creating a runtime.
4. **Connect.** For an interactive shell, run `colab ssh -s SESSION`. Use `colab ssh --help` to confirm options such as `--identity`/`-i`, `--gpu`, `--tpu`, `--high-mem`, and `--rm`. Current upstream docs say the interactive shell starts in `/content`.
5. **Use `--proxy-mode` only for OpenSSH integrations.** Configure an SSH `Host` entry whose `ProxyCommand` invokes `colab ssh --proxy-mode`. Use the absolute path to `colab`, since OpenSSH may invoke the proxy from a non-login shell with a limited PATH. Keep stdout exclusively for the SSH byte stream; do not add wrappers that print banners or diagnostics to stdout.
6. **Manage lifecycle deliberately.** Use `--rm` only when the user wants that runtime stopped on disconnect. Current docs distinguish an auto-created interactive runtime from a reused session; proxy-mode teardown behavior also depends on this flag. For sessions without automatic removal, run `colab stop -s SESSION` after the work and verify with `colab sessions`.
7. **Verify and diagnose.** Confirm the remote shell is responsive and the intended runtime is active. On handshake failures, follow the CLI's status-specific error, check key type/identity, session name, and CLI version. Avoid repeatedly creating VMs to troubleshoot.

## Security guardrails

- Never disclose or upload the private key. The CLI sends the selected public key; private-key material stays local.
- Do not disable host-key verification globally. The upstream dynamic ProxyCommand example disables known-host checking for that alias; scope any such setting narrowly and explain its security tradeoff.
- Treat a Colab runtime as ephemeral, not a persistent SSH server. Do not configure tunnels, web services, proxies, or remote-control workflows that violate Colab policy.
- Do not use `--rm` on a session that must be preserved. Conversely, do not leave disposable sessions running unintentionally.
- Do not silently generate/replace SSH keys or modify `~/.ssh/config`. Preserve existing files and narrowly scope edits to a named host alias.
- Keep SSH output, command logs, and remote files free of credentials. Avoid running untrusted code in a runtime with Drive or cloud credentials.

## Reference

Read `references/ssh-reference.md` for key requirements, command modes, OpenSSH configuration patterns, lifecycle flags, troubleshooting, policy caveats, and source links. Treat upstream documentation and installed `colab ssh --help` as authoritative if behavior changes.
