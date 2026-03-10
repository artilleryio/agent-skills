---
name: convert-artillery-yaml-to-ts
description: >-
  Convert existing Artillery YAML scripts to TypeScript or ESM JavaScript.
  Use when the user wants to migrate Artillery test scripts from YAML to TS/JS for better IDE support, type checking, and composability.
compatibility: Requires Node.js. Input must be an Artillery YAML script.
metadata:
  author: artilleryio
  version: "1.0"
allowed-tools: Bash Read Write Glob Grep
---

# Artillery: Convert YAML scripts to TypeScript/JS

You are converting Artillery YAML test scripts to TypeScript or ESM JavaScript. The conversion is a 1:1 structural mapping — every YAML key maps directly to a JS export. Follow the steps below. Ask the user questions at each decision point marked with DECISION.

## Step 1: Find Artillery YAML files

Glob for `*.yml` and `*.yaml` files in the project. Identify Artillery scripts by checking for `config:` and `scenarios:` as top-level keys. List the files found.

DECISION — If multiple Artillery YAML files exist, ask the user which to convert (or all).

## Step 2: Detect output format

DECISION — Determine whether to output TypeScript or ESM JavaScript:

- If `tsconfig.json` exists, or `.ts` files are present in test directories → output `.ts` with type imports
- If the project is plain JS with no TypeScript setup → output `.mjs`
- If unclear → ask the user

## Step 3: Check for YAML anchors and merge keys

Before converting, scan each YAML file for anchors (`&name`), aliases (`*name`), and merge keys (`<<:`). These are YAML-specific features with no JS equivalent.

If found:
1. Still attempt the conversion — resolve anchors/aliases by inlining the referenced values (duplicate the data at each usage site).
2. Add a comment at the top of the output file: `// WARNING: This file was converted from YAML that used anchors/merge keys. The shared references have been inlined as duplicated values. Review carefully — the conversion may not be accurate.`
3. Warn the user in the output summary which files contained anchors/merge keys and that manual review is needed.

## Step 4: Convert each YAML file

For each Artillery YAML file, read it and convert to a JS/TS module. The mapping is purely structural — YAML values become JS object literals 1:1.

### Top-level key mapping

| YAML key | JS/TS export |
|---|---|
| `config:` | `export const config = { ... }` |
| `scenarios:` | `export const scenarios = [ ... ]` |
| `before:` | `export const before = { ... }` |
| `after:` | `export const after = { ... }` |

### Value mapping rules

- Strings stay strings (quote with double quotes)
- Numbers stay numbers
- Arrays stay arrays
- Nested objects stay objects
- `processor` fields: keep as-is (string reference)
- Artillery template expressions like `{{ authToken }}` stay as literal strings: `"{{ authToken }}"`

### TypeScript output

If outputting `.ts`, add type imports and annotations:

```typescript
import type { Config, Scenario } from "artillery";

export const config: Config = { ... };
export const scenarios: Scenario[] = [ ... ];
```

`before` and `after` do not have dedicated Artillery types — export them untyped.

### ESM JavaScript output

If outputting `.mjs`, no type imports or annotations — just plain `export const`.

### File placement

Write the new file next to the original YAML file, with the same base name but `.ts` or `.mjs` extension.

### Conversion example

This YAML:

```yaml
config:
  target: https://api.example.com
  phases:
    - duration: 60
      arrivalRate: 5
  http:
    timeout: 10
before:
  flow:
    - post:
        url: "/auth/login"
        json:
          username: "testuser"
          password: "testpass"
        capture:
          - json: "$.token"
            as: "authToken"
scenarios:
  - name: "Browse products"
    flow:
      - get:
          url: "/products"
          headers:
            Authorization: "Bearer {{ authToken }}"
```

Becomes this TypeScript:

```typescript
import type { Config, Scenario } from "artillery";

export const config: Config = {
  target: "https://api.example.com",
  phases: [
    { duration: 60, arrivalRate: 5 },
  ],
  http: {
    timeout: 10,
  },
};

export const before = {
  flow: [
    {
      post: {
        url: "/auth/login",
        json: { username: "testuser", password: "testpass" },
        capture: [{ json: "$.token", as: "authToken" }],
      },
    },
  ],
};

export const scenarios: Scenario[] = [
  {
    name: "Browse products",
    flow: [
      {
        get: {
          url: "/products",
          headers: { Authorization: "Bearer {{ authToken }}" },
        },
      },
    ],
  },
];
```

## Step 5: Verify

Try to verify the converted file:

2. **If TypeScript**: run `npx tsc --noEmit <file>` to type-check
3. **If `.mjs`**: run `node --check <file>` to syntax-check
4. **If none of the above work**: skip verification and tell the user to verify manually

## Step 6: Output summary

Tell the user:

- Which files were converted and where the new files are
- The run command: `npx artillery run ./path/to/new-file.ts` (or `.mjs`)
- Original YAML files were left untouched — the user can delete them manually if desired
