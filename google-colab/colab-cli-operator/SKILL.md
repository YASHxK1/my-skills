---
name: colab-cli-operator
description: Operate Google Colab remote runtimes from Linux or macOS using the official google-colab-cli. Use when an AI agent needs to provision CPU/GPU/TPU sessions, run local Python or notebooks remotely, transfer files, mount Drive, inspect logs, or clean up Colab resources.
---

# Google Colab CLI Operator

Use the official `google-colab-cli` (`colab` command) to run workloads on Colab-managed runtimes. The CLI currently supports Linux and macOS; Windows is not supported natively. Read [the bundled command and workflow reference](references/cli-reference.md) when exact commands, flags, or advanced usage are needed.

## Operating procedure

1. **Check availability and version.** Run `colab version` and relevant `colab <command> --help`. Do not assume a flag from memory or an old tutorial; the CLI evolves.
2. **Respect the user's target.** Confirm the requested accelerator, files, output destinations, and whether the user expects a reusable session or one-shot job. Do not silently choose scarce or costly hardware when CPU/default is sufficient.
3. **Authenticate through supported flows.** Use the CLI's OAuth2 flow or existing ADC. Never ask the user to paste credentials or tokens into chat or logs. Distinguish local CLI authentication from `colab auth`, which authenticates code inside the remote VM to Google Cloud services.
4. **Choose the smallest suitable lifecycle.** Prefer `colab run` for an isolated script that should be provisioned and torn down automatically. Use `colab new` when multiple steps, uploads, interactive inspection, or runtime reuse are needed.
5. **Make remote inputs explicit.** `colab exec -f local_script.py` transmits the script contents, but not arbitrary adjacent data or dependencies. Upload required inputs or mount storage. Install packages in the remote session.
6. **Verify outcomes.** Check actual hardware with `colab status`, inspect exit status and logs, and confirm expected files exist locally after download. Do not claim success solely because provisioning succeeded.
7. **Clean up.** Stop manually created sessions when finished. `colab run` normally cleans up unless `--keep` is used; if kept, identify and stop that session. In shell automation, use cleanup traps/finally handling.
8. **Respect Colab limits and policies.** Availability is not guaranteed. Do not use managed runtimes to evade restrictions, host services, mine cryptocurrency, proxy traffic, or perform other prohibited activities. Remote shell/SSH and UI-bypass patterns may be restricted on free use; check Google's current FAQ and terms.

## Safe default workflow

For a staged job, prefer an explicit named session and explicit artifact retrieval:

```bash
colab new -s work --gpu T4
colab install -s work torch transformers
colab upload -s work ./input.csv /content/input.csv
colab exec -s work -f train.py
colab download -s work /content/model.safetensors ./model.safetensors
colab log -s work -o ./run_log.ipynb
colab stop -s work
```

Adjust inputs, packages, paths, and accelerator to the task. Verify exact option spellings via installed CLI help. Use `colab run` instead for a one-shot script that should be automatically cleaned up.

## Safety and reliability

Treat VM storage as ephemeral and download or persist valuable outputs before stopping. Keep secrets out of scripts, notebook outputs, and logs. Inspect paths before destructive remote file operations. Do not install untrusted code in a runtime with Drive or cloud credentials. Do not assume an accelerator request was granted; verify it. If a command fails, inspect status, help, authentication, and logs before creating another session. Avoid repeated provisioning that leaves abandoned sessions.

## Reference

Read `references/cli-reference.md` for installation, command families, authentication, `run` versus `new`, files and Drive, troubleshooting, policy constraints, and upstream links. Prefer installed CLI help and upstream documentation whenever exact behavior may have changed.
