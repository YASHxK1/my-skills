---
name: chrome-headless-operator
description: Use Google Chrome in Headless mode for command-line page rendering, screenshots, PDFs, DOM capture, and browser automation. Use when an AI agent needs to run Chrome without a visible UI in CI, scripts, containers, or server environments, or decide between Chrome CLI, Puppeteer, Selenium, and Headless Shell.
---

# Chrome Headless Operator

Use modern Chrome Headless to run Chrome without a visible UI. Since Chrome 112, unified Headless uses the same Chrome implementation as headful mode. Prefer the normal Chrome binary with `--headless`; use standalone `chrome-headless-shell` only when a workload specifically needs the legacy implementation. Read [the bundled reference](references/headless-reference.md) for flags, automation choices, examples, and troubleshooting.

## Workflow

1. **Choose the right interface.** Use the CLI for a single navigation/capture or DOM serialization. Use Puppeteer or Selenium for multi-step interaction, explicit readiness checks, selectors, screenshots/PDF control, or repeated test suites. Treat old Headless Shell as a distinct legacy binary, not as the default Chrome.
2. **Check the installation.** Locate the Chrome executable (`command -v google-chrome`, `command -v chromium`, or platform-specific path) and check its version. Use flags supported by that installed version.
3. **Run with explicit inputs and outputs.** Specify the target URL, capture type, output path/current working directory, viewport when relevant, and an appropriate timeout. Capture stderr and exit status for diagnosis.
4. **Wait for the right page state.** CLI `--timeout` is a maximum wait before capture, not a guarantee that a JavaScript application is fully ready. For dynamic pages, use Puppeteer/Selenium and wait for a concrete selector, navigation, or application condition.
5. **Keep browser state isolated.** Use a dedicated temporary `--user-data-dir` when a clean profile is needed. Do not reuse or expose a user's logged-in Chrome profile without explicit authorization. Remove temporary profiles after the run if they contain sensitive state.
6. **Protect remote debugging.** Do not expose a DevTools debugging port to an untrusted network. Bind it to localhost when necessary, restrict access, and shut the browser down after automation.
7. **Verify artifacts.** Confirm the command exited successfully and that the screenshot/PDF/output exists, is nonempty, and corresponds to the requested page. Mention pages that require sign-in, block automation, or render incompletely.
8. **Respect access controls.** Do not use Headless to evade authentication, paywalls, CAPTCHAs, rate limits, or other site restrictions. Follow the website's terms and applicable policy.

## Quick CLI patterns

```bash
# Serialize the post-script DOM to stdout
chrome --headless --dump-dom 'https://example.com/'

# Capture a viewport screenshot
chrome --headless --screenshot --window-size=1280,800 'https://example.com/'

# Save a PDF without printed header/footer
chrome --headless --print-to-pdf=page.pdf --no-pdf-header-footer 'https://example.com/'
```

Use an explicit output directory or run from a controlled working directory. Consult the detailed reference for quoting, path behavior, timeout/virtual-time options, headless shell differences, and automation examples.

## Core distinctions

- **Modern Chrome Headless:** Use `chrome --headless`; unified with Chrome's regular browser implementation since Chrome 112.
- **Headless Shell:** Legacy separate implementation, available as standalone `chrome-headless-shell` since Chrome 132. Use only for compatibility or a deliberate shell-specific need.
- **Puppeteer/Selenium:** Use when deterministic interaction and explicit waits are required. CLI capture flags are not a complete substitute for browser automation.

## Reference

Read `references/headless-reference.md` for exact CLI commands, options, automation patterns, screen configuration, security, and official source links. Check current Chrome documentation and local `chrome --help` when flags or behavior may vary by release.
