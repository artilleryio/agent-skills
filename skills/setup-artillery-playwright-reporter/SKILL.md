---
name: setup-artillery-playwright-reporter
description: >-
  Set up the Artillery Playwright reporter in an existing Playwright E2E test
  suite. Installs @artilleryio/playwright-reporter, configures it in
  playwright.config.ts, sets up Artillery Cloud API key, and runs the suite to
  verify reporting works. Use when the user wants to add Artillery Cloud
  reporting to their Playwright tests, monitor E2E test results in a dashboard,
  or integrate Playwright with Artillery Cloud.
compatibility: Requires Node.js and an existing Playwright E2E test suite.
metadata:
  author: artilleryio
  version: "1.0"
allowed-tools: Bash Read Write Glob Grep
---

# Artillery: Set up the Playwright reporter

You are setting up the Artillery Playwright reporter (https://artillery.io) for an existing Playwright E2E test suite. Follow the steps below. Ask the user questions at each decision point marked with DECISION.

## Step 1: Detect environment

Examine the project to determine:

- **Playwright config**: Look for `playwright.config.ts` (or `playwright.config.js`). If multiple configs exist, proceed to DECISION below.
- **Package manager**: Look for `package-lock.json` (npm), `pnpm-lock.yaml` (pnpm), `yarn.lock` (yarn), `bun.lockb` (bun).
- **Monorepo vs single project**: Look for `turbo.json`, `pnpm-workspace.yaml`, `lerna.json`, or `workspaces` in `package.json`. Determine which workspace contains the Playwright test suite.
- **Existing reporter config**: Check if the `playwright.config.ts` already has a `reporter` section.
- **Secret management patterns**: Look for `.env` files, `.env.local`, `.env.example`, or other secret management approaches used in the project.

DECISION — If multiple `playwright.config.ts` files exist, ask the user which one to target.

## Step 2: Install the reporter

Install `@artilleryio/playwright-reporter` as a dev dependency using the detected package manager. In a monorepo, run the install in the workspace that contains the Playwright test suite (e.g. `pnpm add -D --filter <workspace> @artilleryio/playwright-reporter`).

## Step 3: Configure the reporter in playwright.config.ts

Add `@artilleryio/playwright-reporter` to the beginning of the `reporter` array in `playwright.config.ts`. If no `reporter` section exists yet, create one. Merge with any existing reporters — do not remove them.

```typescript
export default defineConfig({
  reporter: [
    ['@artilleryio/playwright-reporter', { name: 'My E2E Suite' }],
    ['html', { open: 'never' }], // keep any existing reporters
    ['dot'],
  ],
});
```

The reporter accepts an optional `name` property to label the test suite in Artillery Cloud. Use a descriptive name based on the project or workspace name.

## Step 4: Set up Artillery Cloud API key

The reporter needs an `ARTILLERY_CLOUD_API_KEY` environment variable to send results to Artillery Cloud.

DECISION — Ask the user:

1. **Do you have an Artillery Cloud account?** If not, sign up for free at https://app.artillery.io and grab an API key from Settings > API Keys.
2. **How should the API key be stored?** Based on the project's existing patterns:
   - If `.env` / `.env.local` files are used: add `ARTILLERY_CLOUD_API_KEY=<key>` to the appropriate `.env` file. Verify it is listed in `.gitignore`.
   - If no `.env` pattern exists: instruct the user to export it in their shell:
     ```bash
     export ARTILLERY_CLOUD_API_KEY=<their-key>
     ```
   - If another secret management approach is used in the project: ask the user how to proceed.

## Step 5: Run the test suite

Run the Playwright test suite to verify the reporter works. Use the project's existing test command — check `package.json` scripts for a Playwright test command (e.g. `npm test`, `npm run test:e2e`). If none exists, use:

```bash
npx playwright test
```

Once complete, share the following with the user:

- The Artillery Cloud URL printed in the terminal links to the live test report.
- Existing reporters (HTML, dot, JSON, etc.) continue to work alongside the Artillery reporter.
- Every subsequent `npx playwright test` run will automatically report to Artillery Cloud as long as the API key is set.
