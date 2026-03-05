# Artillery Agent Skills

A collection of AI agent skills for Artillery. All skills follow the [Agent Skills](https://agentskills.io/) format and use Tessl for automated linting & evaluation.

## Available skills

### `setup-artillery-cli-for-load-testing` - quickstart onboarding for load testing

See Tessl's report: [tessl.io/setup-artillery-cli-for-load-testing](https://tessl.io/registry/skills/github/artilleryio/agent-skills/setup-artillery-cli-for-load-testing/review)

**Use when**: adding load testing with Artillery to a new JS/TS-based project.

**Outcomes**: Artillery CLI installed in your current project (or an appropriate monorepo workspace), Artillery Cloud reporting integration is set up, and a simple initial load testing script is created - for a HTTP API or a web app (with Playwright).

### `setup-artillery-playwright-reporter` - add Artillery Cloud reporting to Playwright E2E tests

See Tessl's report: [tessl.io/setup-artillery-playwright-reporter](https://tessl.io/registry/skills/github/artilleryio/agent-skills/setup-artillery-playwright-reporter/review)

**Use when**: adding Artillery Cloud reporting to an existing Playwright E2E test suite.

**Outcomes**: `@artilleryio/playwright-reporter` installed, configured in `playwright.config.ts`, Artillery Cloud API key set up, and test suite verified to report results to the Artillery Cloud dashboard.
