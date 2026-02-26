# Trace, Reporting & CI Policy (Placeholder)

> Status: DRAFT  
> Owner: TODO(@team/owner) • Last updated: TODO(YYYY-MM-DD)

## 1) Objectives
- Fast root-cause via rich artifacts (trace, video, logs).
- Stable CI with actionable signals and clear ownership.

## 2) Tracing Policy
- **TS/JS (Playwright Test)**:
  - Local: tracing `off` by default; enable ad-hoc if needed.
  - CI: `trace: 'on-first-retry'` for all projects; always on for critical suites (TODO:list).
- **Python (pytest + Playwright)**:
  - Start tracing at test start; **stop & save only on failure/retry** via fixture.
  - Artifact path: `artifacts/traces/${TEST_NAME}.zip`
- **Screenshots/Videos**:
  - On failure (default). Always-on for exploratory suites (TODO:list).

## 3) Artifact Retention
- Default retention: **7 days CI**, **30 days for critical suites**.
- Nightly builds: store only last **N=5** runs.
- Redact PII; encryption at rest in CI storage.
- TODO: bucket/URL and access policy.

## 4) Reporting Stack
- **Primary**: Playwright HTML report (TS/JS).
- **Interop**: JUnit XML (TS/JS & Python) for CI test summaries.
- **Optional**: Allure for unified cross-stack reporting.
- Single index page linking web (Playwright) + mobile (Appium) results.

## 5) CI Matrix & Concurrency
- Browsers: Chromium, Firefox, WebKit.
- OS: Ubuntu main; Windows/macOS nightly (smoke).
- Concurrency: `max-workers = CPU cores` (cap at TODO N).
- Sharding: by file count; balance shards using CI plugin or custom script.

### Example: GitHub Actions (TS/JS)
```yaml
name: e2e
on: [push, workflow_dispatch]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        project: [chromium, firefox, webkit]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test --project=${{ matrix.project }}
      - run: npx playwright show-report
        if: failure()
      - name: Upload HTML report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: report-${{ matrix.project }}
          path: playwright-report
```

### Example: Pytest (Python) job
```yaml
jobs:
  pytest:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.11' }
      - run: pip install -r requirements.txt
      - run: python -m playwright install --with-deps
      - run: pytest -q --junitxml=junit.xml
      - name: Upload artifacts
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: python-artifacts
          path: |
            junit.xml
            **/*-trace.zip
```

## 6) Quality Gates & SLOs
- Build must fail if:
  - Pass rate < **X%** (TODO) or new failures > **N**.
  - Flaky rate (retries succeeded) > **Y%** week-over-week.
- Slowest test warning: any test > **Z sec** (TODO) logged for triage.

## 7) Flaky Management
- Auto-quarantine label: `@flaky` + issue auto-filed to owner.
- Triage SLA: within **24–48h**; RCA note required.
- Exit criteria from quarantine: **3 consecutive green runs**.

## 8) Failure Taxonomy (label at source)
- `infra-*` (runners, network, timeouts)
- `product-*` (backend bug, feature flag, data)
- `test-*` (selector, timing, assertions, data setup)

## 9) Ownership & Notifications
- Ownership file: `CODEOWNERS` + per-suite OWNERS.md.
- Notifications: Slack/Email on red builds; on-call schedule link.
- Escalation path: TODO(document)

## 10) Appium Device Farms
- Default device profiles: TODO(list models/OS)
- Parallel runs per suite: TODO(number)
- Video/Logcat collection: enabled for failures.

---

## References (fill in)
- Playwright docs: setup, test retries, trace viewer.
- Appium docs: Python client, capabilities.
- Internal dashboards/links: TODO
