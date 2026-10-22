# Chrome Headless Reference

Use current Chrome documentation and the installed binary's `--help` as sources of truth. Official overview: https://developer.chrome.com/docs/automation-and-testing/headless

## Mode selection and version notes

Modern Headless is enabled with `--headless`. Starting in Chrome 112, it creates platform windows without displaying them and shares Chrome's regular implementation. Since Chrome 132, the old Headless implementation is available only as the standalone `chrome-headless-shell` binary. In Puppeteer, `headless: true` selects modern Headless; `headless: 'shell'` selects Headless Shell; `headless: false` runs headful Chrome. Selenium can set Chrome option `--headless`.

Prefer modern Headless by default. Select Headless Shell only when a compatibility constraint or benchmark specifically requires it. Do not copy flags from old Headless articles without checking current docs.

## CLI capture commands

Use the Chrome executable available on the machine. Common executable names include `chrome`, `google-chrome`, and `chromium`; paths vary by operating system and installation method.

```bash
# DOM after HTML parsing and script execution, serialized to stdout
chrome --headless --dump-dom 'https://example.com/'

# Screenshot; writes screenshot.png in the current working directory
chrome --headless --screenshot 'https://example.com/'

# Screenshot with explicit viewport
chrome --headless --screenshot --window-size=1280,800 'https://example.com/'

# PDF; default output.pdf in current working directory
chrome --headless --print-to-pdf 'https://example.com/'

# Explicit PDF path, with headers/footers omitted
chrome --headless --print-to-pdf=/tmp/page.pdf --no-pdf-header-footer 'https://example.com/'
```

Chrome docs also show `--print-to-pdf=output.pdf`. Some docs/examples use whitespace around `=` due to formatting; prefer conventional `--flag=value` or `--flag value` syntax and confirm with the installed Chrome build.

`--dump-dom` is not equivalent to downloading source HTML: Chrome parses markup, runs page scripts, then serializes the DOM. It prints to stdout, so redirect stdout deliberately when saving it.

### Waiting and time-dependent pages

`--timeout=5000` sets the maximum wait in milliseconds before `--dump-dom`, `--screenshot`, or `--print-to-pdf` captures, even if loading continues. It is a time limit, not an app-readiness assertion.

`--virtual-time-budget=42000` can fast-forward time-dependent page code such as timers for capture. This is useful for deterministic timer-based demos, but does not mean real network work has completed or that every app can be reliably fast-forwarded.

For JS-heavy apps, prefer an automation library and wait for a meaningful condition:

```js
await page.goto(url, { waitUntil: 'domcontentloaded' });
await page.waitForSelector('[data-test="ready"]');
await page.screenshot({ path: 'page.png' });
```

Choose a selector that actually indicates the content requested. `networkidle` can be inappropriate on apps with long-polling, analytics, or persistent network connections.

### Chrome-internal pages

`--allow-chrome-scheme-url` is required to access `chrome://` URLs and is documented as available since Chrome 123:

```bash
chrome --headless --allow-chrome-scheme-url --dump-dom chrome://gpu
```

Do not add this flag for ordinary web pages.

## Programmatic automation

Use Puppeteer for Node.js browser automation. Its default headless option uses modern Headless. A minimal lifecycle pattern:

```js
import puppeteer from 'puppeteer';

const browser = await puppeteer.launch({ headless: true });
try {
  const page = await browser.newPage();
  await page.setViewport({ width: 1280, height: 800 });
  await page.goto('https://example.com/', { waitUntil: 'domcontentloaded' });
  await page.screenshot({ path: 'page.png', fullPage: true });
} finally {
  await browser.close();
}
```

Use `puppeteer-core` only when Chrome installation/selection is managed separately and provide the executable path where needed. For Selenium WebDriver, add `--headless` to Chrome options and use the language binding's explicit-wait API. Use one automation framework consistently unless there is a specific need to combine them.

## Viewports, screenshots, and virtual displays

For simple page screenshots, use `--window-size=WIDTH,HEIGHT`. Modern Headless also supports virtual screen configuration with `--screen-info`, and dynamic screen add/remove through Chrome DevTools Protocol. The official virtual-screen documentation says this functionality is stable starting in Chrome 142. Use it for multi-monitor, orientation, scaling, and fullscreen scenarios; do not confuse virtual display/screen configuration with a page viewport size.

For full-page screenshots or precise multi-screen behavior, prefer Puppeteer/CDP rather than assuming `--screenshot` captures the entire scrollable page. Verify dimensions of the produced artifact.

## Remote debugging and security

`--remote-debugging-port=9222` enables the Chrome DevTools Protocol endpoint. This endpoint can control the browser and access page data, so treat it as highly privileged. Never expose it on a public or untrusted interface. If needed, bind it to loopback, firewall it, avoid placing credentials in the debugged profile, and terminate Chrome after use. Use an isolated temporary user-data directory for automated sessions that need persistence:

```bash
chrome --headless \
  --user-data-dir=/tmp/chrome-profile-job \
  --remote-debugging-port=9222 \
  'https://example.com/'
```

Use a unique profile directory per concurrent browser process. Do not reuse a personal profile or copy session cookies unless explicitly authorized and required.

## Troubleshooting

- **Executable not found:** Locate Chrome/Chromium and use the full path. Do not assume the command name is identical across platforms.
- **Screenshot/PDF missing:** Check exit status, current working directory, writable output path, stderr, and whether the page navigation failed.
- **Capture is blank or incomplete:** Increase the timeout for slow pages or switch to Puppeteer/Selenium and wait for the specific content. A fixed timeout alone cannot prove readiness.
- **Viewport mismatch:** Set `--window-size=WIDTH,HEIGHT`, and separately set device scale factor/viewport via the automation framework if needed.
- **PDF headers unexpected:** Use `--no-pdf-header-footer`; older Chrome builds may use the legacy `--print-to-pdf-no-header` flag.
- **Sandbox/container launch failure:** Diagnose environment and permissions first. Do not blindly add `--no-sandbox`; it disables a core security boundary. If an isolated container truly requires it, understand the threat model and never use it to run untrusted browsing with privileged host access.
- **Chrome process left running:** Ensure automation always closes the browser in a `finally`/`try...finally` path; inspect and terminate only the process started for the job.
- **Headless differs from headful:** Confirm modern Headless versus Headless Shell, Chrome version, viewport, fonts, locale, timezone, GPU/compositor environment, and profile state.

## Sources

- Official Headless overview: https://developer.chrome.com/docs/automation-and-testing/headless
- CLI reference: https://developer.chrome.com/docs/automation-and-testing/headless-cli
- Headless Shell article: https://developer.chrome.com/docs/automation-and-testing/headless-chrome-shell
- Virtual headless screens: https://developer.chrome.com/docs/automation-and-testing/headless-screen-config
- Puppeteer headless modes: https://pptr.dev/guides/headless-modes
- Selenium Headless announcement/context: https://www.selenium.dev/blog/2023/headless-is-going-away/
