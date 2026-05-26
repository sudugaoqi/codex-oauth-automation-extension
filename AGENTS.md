# Repository Guidelines

## Project Structure & Module Organization
`background.js` is the Manifest V3 service-worker entry shell. Keep new orchestration in `background/` or `background/steps/` instead of growing the root entry again. Put page automation in `content/`, side panel UI code in `sidepanel/`, shared step metadata and seed data in `data/`, SMS adapters in `phone-sms/providers/`, local helper scripts in `scripts/`, regression coverage in `tests/`, and longer design or usage notes in `docs/`. If step titles, order, or keys change, update both `data/step-definitions.js` and `background/steps/registry.js`.

## Build, Test, and Development Commands
There is no bundling step. Load the repository root as an unpacked extension from `chrome://extensions`.

- `npm test` runs the full JavaScript regression suite with Node's built-in test runner.
- `node --test tests/background-step-registry.test.js` runs one focused JavaScript suite while iterating.
- `python3 -m unittest tests/hotmail_helper_logging_test.py` runs the Python helper regression test.
- `python3 scripts/hotmail_helper.py` starts the local Hotmail helper used by some mail flows.

## Coding Style & Naming Conventions
Follow the existing plain-JavaScript style: 2-space indentation, semicolons, single quotes, and descriptive camelCase identifiers. Keep `background.js` and `sidepanel/sidepanel.js` as wiring layers; shared logic belongs in smaller modules. Name new step files semantically, such as `fetch-login-code.js`, not `step8.js`. For step-aware logs, pass structured metadata like `{ step, stepKey }` instead of encoding step numbers into log text.
OpenSpec proposal, design, spec, and tasks documents should default to Chinese prose. Keep any schema-required labels unchanged when the tool depends on exact wording.

## Testing Guidelines
Every behavior change should add or adjust regression tests, especially around step routing, provider-specific branches, stop/retry handling, and shared utilities. JavaScript tests live in `tests/*.test.js`; Python helper tests follow `tests/*_test.py`. When a change adds, removes, or renames files or alters the flow, update `项目文件结构说明.md` and the relevant docs in `docs/`.

## Commit & Pull Request Guidelines
The visible history is sparse (`Initial snapshot`), so use short, specific imperative commit subjects such as `Add 2925 cooldown regression test` and avoid vague messages like `update`. PRs should target `dev`, sync with the latest `origin/dev` before opening, and describe the real user-facing change in natural Chinese. Include screenshots for `sidepanel/` UI changes and list the exact tests you ran. Never commit live OAuth URLs, API keys, mailbox credentials, or exported account data.
