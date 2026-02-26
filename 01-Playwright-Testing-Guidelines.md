# Playwright Testing Guidelines (Placeholder)

> Status: DRAFT  
> Owner: TODO(@team/owner) • Last updated: TODO(YYYY-MM-DD)

## 1) Scope & Tech Stack
- Default runtime: **TypeScript/JavaScript with Playwright Test**.
- Python (pytest + Playwright) **on request**.
- Native apps: **Appium (Python)** for Android/iOS; unify reporting with web tests.

## 2) Repository Layout (TS/JS default)
```
tests/
  e2e/
  components/
playwright.config.ts
package.json
```
- **Python layout (when used)**: `tests/`, `conftest.py`, `requirements.txt`.

## 3) Coding & Naming Conventions
- Test files: `*.spec.ts` / `*.test.ts`
- Test names: behavior-driven, present tense, clear intent.
- Page Objects: `PageName.page.ts`
- TODO: define linters/formatters (ESLint/Prettier rules)

## 4) Selector Strategy (Order of preference)
1. **getByRole** with accessible name (stable, a11y-friendly)
2. **data-testid** (stable when a11y not feasible)
3. **locator** with explicit constraints
- Prohibited: brittle CSS chains/xpaths unless justified.
- TODO: standardize `data-testid` naming scheme.

## 5) Page Object Model (POM)
- Keep POM thin: expose **intentful methods** (e.g., `loginAs(user)`), hide selectors.
- One POM per page/feature; compose where sensible.
- Anti-patterns: test logic inside POM, deep inheritance.

## 6) Fixtures & Auth
- TS/JS: use `test.extend()` with `storageState` for shared sessions.
- Python: session-scoped fixture with `storage_state` equivalent via context.
- Rotate credentials via secrets (NO hardcoding).
- TODO: define `auth` bootstrap & refresh policy.

## 7) Test Data & Environments
- Prefer **API seeding** over UI.
- Idempotent seeds; teardown on need.
- Env config via `process.env` (TS/JS) or `os.environ` (Python).
- TODO: document env matrix (dev/stage/prod-like).

## 8) Stability Defaults
- **Retries**: 2 (CI) / 0 (local) — adjust per team SLA.
- **Timeouts**: prefer expectation timeouts over global.
- **Parallelism**: enable per file; shard on CI by file count.
- TODO: define max test duration & slow test threshold.

## 9) Tracing & Artifacts
- TS/JS: `trace: 'on-first-retry'` (CI), `off` (local by default).
- Python: trace on failure/retry via **fixture start/stop** (see sample below).
- Screenshots/videos: on failure.
- Retention: see Trace/Reporting policy doc.

## 10) Reporting
- TS/JS: built-in HTML + JUnit/Allure (optional).
- Python: HTML/Allure with unified index for mixed stacks.
- TODO: link to org’s reporting dashboard.

## 11) CI Guidelines
- Browsers: Chromium + Firefox + WebKit (smoke/full tiers).
- OS matrix: TODO(select) (e.g., Ubuntu main; Windows/macOS nightly).
- Sharding: per file; balanced shards.
- Flaky quarantine: see policy.

## 12) Security & Compliance
- No secrets in code or logs.
- Scrub PII from artifacts.
- TODO: retention periods per environment.

## 13) PR Review Checklist
- Clear test intent + stable selectors
- No sleeps; proper waits/expectations
- POM boundaries respected
- Artifacts enabled on CI failures
- Docs & links updated

---

### Samples

**TS/JS — `playwright.config.ts`**
```ts
import { defineConfig } from '@playwright/test';
export default defineConfig({
  retries: process.env.CI ? 2 : 0,
  use: { baseURL: process.env.BASE_URL, trace: process.env.CI ? 'on-first-retry' : 'off' },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
    { name: 'firefox',  use: { browserName: 'firefox' } },
    { name: 'webkit',   use: { browserName: 'webkit' } },
  ],
  reporter: [['html'], ['junit', { outputFile: 'junit.xml' }]],
});
```

**Python — tracing via fixture**
```python
# conftest.py
import pytest
from playwright.sync_api import sync_playwright

@pytest.fixture(scope="session")
def browser():
    with sync_playwright() as p:
        b = p.chromium.launch(headless=True)
        yield b
        b.close()

@pytest.fixture
def page(browser, request, tmp_path):
    ctx = browser.new_context()
    pg = ctx.new_page()
    # Start tracing
    ctx.tracing.start(screenshots=True, snapshots=True, sources=True)
    yield pg
    # Stop tracing on failure only
    if request.node.rep_call.failed:
        trace_zip = tmp_path / f"{request.node.name}-trace.zip"
        ctx.tracing.stop(path=str(trace_zip))
    else:
        ctx.tracing.stop()
    ctx.close()

# hook: to know test outcome inside fixture
def pytest_runtest_makereport(item, call):
    if "rep_" + call.when not in item.__dict__:
        setattr(item, "rep_" + call.when, call)
```

**Appium (Python) — minimal caps**
```python
from appium import webdriver
caps = {
  "platformName": "Android",
  "automationName": "UiAutomator2",
  "appPackage": "com.example.app",
  "appActivity": ".MainActivity",
  "newCommandTimeout": 120
}
driver = webdriver.Remote("http://127.0.0.1:4723/wd/hub", caps)
```

---

## TODOs
- [ ] Fill team owners, env matrix, retention values, flaky SLA.
- [ ] Adopt selector naming convention.
- [ ] Link CI pipelines and dashboards.
