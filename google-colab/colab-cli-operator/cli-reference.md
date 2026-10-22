# Colab CLI Command Reference

Use the official repository as the source of truth: https://github.com/googlecolab/google-colab-cli. The project currently documents Linux and macOS support. Check the installed binary's `--help` because flags and hardware options can change.

## Installation and help

```bash
uv tool install google-colab-cli
# or
pip install google-colab-cli

colab version
colab --help
colab <command> --help
```

## Global options and authentication

The documented global options include:

- `--auth {oauth2,adc}`: Authentication provider; repository README currently documents `adc` as default.
- `-c, --client-oauth-config PATH`: OAuth client configuration path.
- `--config PATH`: Local session metadata file.

OAuth2 uses an authorization URL plus pasted authorization code. ADC uses Google Application Default Credentials. The project's auth document states that user ADC credentials may need scopes including `openid`, `cloud-platform`, `userinfo.email`, and `colaboratory`. Follow live `colab --help` and upstream auth docs if auth behavior changes.

CLI auth is for calling Colab's session backend. `colab auth -s SESSION` is a separate flow to authenticate the remote runtime for Google Cloud services. Never echo, log, or commit credentials.

## Command families

### Sessions

```bash
colab new [-s NAME] [--gpu GPU | --tpu TPU] [--high-mem]
colab sessions
colab status [-s NAME]
colab restart-kernel [-s NAME]
colab stop [-s NAME]
colab url [-s NAME] [--open]
```

Supported accelerators documented in the README: GPU T4, L4, G4, H100, A100; TPU v5e1 and v6e1. Resource availability and account entitlements vary. `--high-mem` is only supported for eligible shapes/plans; L4 and TPU runtimes are documented as ignoring it. Verify actual machine details using `colab status`.

### Execution

```bash
colab exec [-s NAME] [-f LOCAL_PY_OR_IPYNB] [--output-image PATH]
cat script.py | colab exec -s NAME
colab repl [-s NAME]
colab console [-s NAME]
colab ssh [-s NAME] [--proxy-mode] [-i KEY]
```

`colab exec -f` sends the local script or notebook content to the remote kernel; this does not automatically upload its other local data or dependencies. Provide inputs separately. The current execution design documents a default timeout of 30 seconds and configurable `--timeout`; confirm actual syntax with `colab exec --help`.

`repl` and `console` are interactive and ordinarily need a local TTY. `console`/`ssh` may be restricted by Colab policy on some plans even though commands exist.

### One-shot jobs

```bash
colab run [--gpu GPU | --tpu TPU] [--high-mem] [--keep] SCRIPT [ARGS...]
```

`run` provisions a fresh runtime, executes a local Python script and normally tears the VM down. It forwards script arguments as `sys.argv` and propagates script failure status. `--keep` preserves the session; use it only when inspection/reuse is needed, and stop the session afterward. The current documented timeout default is 30 seconds. A GPU example:

```bash
colab run --gpu T4 train.py --epochs 2
```

### Files

```bash
colab ls [-s NAME] [PATH]
colab upload [-s NAME] LOCAL REMOTE
colab download [-s NAME] REMOTE LOCAL
colab rm [-s NAME] PATH
colab edit [-s NAME] PATH
```

Confirm local and remote paths carefully. Download important artifacts before stopping the session. Validate remote deletion paths before using `rm`.

### Runtime utilities

```bash
colab install [-s NAME] [-r REQUIREMENTS_FILE | PACKAGE...]
colab drivemount [-s NAME] [PATH]
colab auth [-s NAME]
colab log [-s NAME] [-n N] [-o OUTPUT]
colab usage
colab pay
colab update [--install]
colab version
```

`install` installs packages remotely, using `uv` where available with a documented pip fallback. `drivemount` defaults to `/content/drive`. Logs can be exported as `.ipynb`, `.md`, `.txt`, or `.jsonl`. Inspect logs for secrets before sharing. `usage` reports compute-unit information where available. `pay` opens Colab's subscription page; do not initiate a purchase without user authorization and the appropriate payment workflow.

## Choosing `run` or `new`

Choose `run` for a one-shot local Python script where automatic provisioning and cleanup are desired. Choose `new` when the work needs several staged commands, uploads, installation, interactive debugging, notebook execution, or reuse of one session. Explicit named sessions help prevent operating on the wrong runtime.

## Safe multi-step pattern

```bash
set -euo pipefail
SESSION=work
cleanup() { colab stop -s "$SESSION" >/dev/null 2>&1 || true; }
trap cleanup EXIT

colab new -s "$SESSION" --gpu T4
colab install -s "$SESSION" -r requirements.txt
colab upload -s "$SESSION" ./input.csv /content/input.csv
colab exec -s "$SESSION" -f ./train.py
colab download -s "$SESSION" /content/model.bin ./model.bin
colab log -s "$SESSION" -o ./execution.ipynb
```

Adjust to supported flags in the installed version. Do not use `trap` as proof cleanup worked: after errors, inspect `colab sessions`, then stop any remaining named session explicitly.

## Colab constraints

Google says Colab resources are not guaranteed or unlimited, and runtimes can terminate after inactivity or maximum lifetime. Managed runtimes prohibit abuse such as crypto mining, hosting services, proxying, or bypassing anti-abuse restrictions. Google's FAQ also identifies remote control such as SSH shells and bypassing notebook UI as disallowed on free use in specified circumstances without a positive compute-unit balance. Recheck the current FAQ and terms: https://research.google.com/colaboratory/faq.html

## Upstream references

- Announcement: https://developers.googleblog.com/introducing-the-google-colab-cli/
- Repository README and current command index: https://github.com/googlecolab/google-colab-cli
- Execution design: https://github.com/googlecolab/google-colab-cli/blob/main/docs/02_execution_and_interactive.md
- Authentication and utilities: https://github.com/googlecolab/google-colab-cli/blob/main/docs/04_automation_and_utility.md
- `run` design: https://github.com/googlecolab/google-colab-cli/blob/main/docs/05_run_command.md
- Colab FAQ and restrictions: https://research.google.com/colaboratory/faq.html
