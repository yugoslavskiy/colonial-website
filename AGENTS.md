# AGENTS.md

This file is guidance for coding agents working in `attack-website`.

## Scope

- Applies to the repository root.
- There was no existing `AGENTS.md` to preserve.
- No Cursor rules were found in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions were found in `.github/copilot-instructions.md`.
- Primary human docs are `DEVELOPMENT.md`, `README.md`, `test/README.md`, and `CONTRIBUTING.md`.

## Repo Shape

- Root Python build system generates the static site via `update-attack.py`.
- `attack-search/` is a separate Node/CommonJS project for the search bundle.
- `attack-style/` is a separate Node/Sass project for CSS output.
- `attack-theme/` contains Jinja templates, static assets, and legacy browser JS.
- `modules/` contains Python modules that generate ATT&CK site content.
- `test/` provides an Nginx Docker environment for validating the built site.

## Environment Expectations

- Python 3 is required for the main build.
- Node.js and npm are required for `attack-search/` and `attack-style/`.
- Docker is the preferred way to validate the final static output in an Nginx-like environment.
- CI currently uses Python `3.13` and Node `18.x` in `.github/workflows/gh-pages.yml`.

## High-Value Commands

Run commands from the repo root unless a subdirectory is called out.

### Install

- Python deps: `python3 -m pip install -r requirements.txt`
- Search deps: `cd attack-search && npm ci`
- Style deps: `cd attack-style && npm ci`

### Build

- Main website build: `python3 update-attack.py --attack-brand --extras --no-test-exitstatus`
- Search bundle: `cd attack-search && npm run build`
- Search dev bundle: `cd attack-search && npm run build:dev`
- Copy built search bundle into site output: `cd attack-search && npm run copy`
- Style build: `cd attack-style && npm run build`
- Style build + copy into theme static assets: `cd attack-style && npm run build-copy`

### Local Validation

- Full local site validation follows `DEVELOPMENT.md` and `test/README.md`.
- Build site output first, then serve `output/` through the Docker test image.
- Test container build: `cd test && docker build -t attack-website-test .`
- Test container run: `cd test && docker run -p 80:80 -v $(pwd)/../output:/workspace attack-website-test`
- Helper script: `cd test && ./run_test.sh`

### Lint And Format

- Python lint (configured, not wired into CI): `ruff .`
- Python format: `black .`
- Python import sort: `isort .`
- Search lint: `cd attack-search && npm run lint`
- Search lint autofix: `cd attack-search && npm run lint:fix`
- Style lint: `cd attack-style && npm run lint`

### Tests

- Search tests: `cd attack-search && npm test`
- Single Jest test file: `cd attack-search && npm test -- __tests__/search-service.test.js`
- Alternate single Jest file: `cd attack-search && npx jest __tests__/search-service.test.js`
- Main Python-driven site tests run through the build script, not `pytest`.
- Run specific site test categories: `python3 update-attack.py -m tests -t size`
- Multiple site test categories: `python3 update-attack.py -m tests -t links external_links citations`

### Important Command Notes

- There is no root `package.json`, `Makefile`, or single universal test runner.
- CI clearly builds the site and search bundle, but does not currently enforce Jest, ESLint, Stylelint, Ruff, Black, or type checks.
- For Python-side testing, the narrowest supported scope is a named category (`size`, `links`, `external_links`, `citations`), not an individual test file.
- Preferred production-like validation is Nginx via Docker, not Pelican's built-in dev server.

## Source Of Truth

- Follow existing file-local conventions before applying generic preferences.
- Treat `pyproject.toml`, `attack-search/.eslintrc`, and `attack-style/.stylelintrc.json` as authoritative style configs.
- Treat `DEVELOPMENT.md` and `.github/workflows/gh-pages.yml` as authoritative for build workflow.
- In templates, respect comments that mark generated files or source-of-truth files.
- Example: `attack-theme/templates/general/base-template.html` explicitly says to edit `base-template.html`, not generated `base.html`.

## Python Style

- Use 4-space indentation.
- Match Black formatting and the configured 120-character line length.
- Keep imports grouped as standard library, third-party, then local imports.
- Use `isort` with the Black profile when reorganizing imports.
- Prefer `snake_case` for variables, functions, and module names.
- Reserve `UPPER_CASE` for constants or config-like values.
- Keep modules function-oriented and simple; the codebase uses few classes outside JS.
- Type hints are not a dominant convention here; do not introduce a large typing layer unless the touched area already uses it.
- Python type hints are desired to be added over time.
- Docstrings are present for many public helpers; follow surrounding patterns instead of adding blanket documentation everywhere.
- Python docstring format follow numpy conventions

## Python Error Handling

- Raise explicit argument validation errors when input is invalid.
- Preserve existing CLI behavior in `update-attack.py`; it is the operational entry point.
- Avoid silent failures in build code unless the surrounding module already degrades intentionally.
- When editing build logic, prefer predictable failures with useful messages over hidden fallbacks.

## JavaScript Style

- In `attack-search/`, use CommonJS (`require`, `module.exports`) unless the file already uses something else.
- Keep local imports consistent with surrounding code; many files include `.js` extensions intentionally for webpack resolution.
- Prefer single quotes in `attack-search/` unless the local file already differs.
- Semicolons are the norm in `attack-search/`; keep them.
- Use `camelCase` for variables and functions, `PascalCase` for classes.
- Modern JS features are acceptable in `attack-search/` (async/await, private methods, optional chaining, nullish coalescing).
- Favor small, explicit DOM interactions over framework-style abstractions; this repo is not React-based.
- In legacy `attack-theme/static/scripts/`, preserve the file's existing style rather than forcing `attack-search/` conventions into older code.

## JavaScript Error Handling And Testing

- For async startup flows, follow the existing `try/catch` and `.catch()` patterns.
- Log actionable errors with `console.error` or `console.debug` where the surrounding code already does so.
- Prefer graceful UI degradation for browser capability issues instead of crashing the page.
- Jest tests live in `attack-search/__tests__/` and usually use `describe`, `test` or `it`, mocked globals, and fixture files.
- When adding tests, follow the existing jsdom/jQuery mocking style rather than introducing a new browser test stack.

## Templates, HTML, And Content Generation

- Jinja templates typically place `set` statements and imports at the top.
- Preserve existing macro usage patterns instead of inlining repeated HTML.
- Do not edit generated artifacts if the template comments point to a source template.
- Keep ATT&CK branding, banner, and version placeholders intact unless the task is explicitly about site configuration.
- Be careful with path handling and generated `index.html` semantics; many helpers normalize trailing slashes.

## SCSS And Styling

- `attack-style/` uses Sass modules with `@use`, not legacy `@import`.
- Preserve the layered structure: `abstracts/`, `base/`, `layout/`, `components/`.
- Import order matters because variables/functions are dependencies for later files.
- Use lowercase, hyphenated naming for classes and partial filenames.
- Reuse existing Sass variables, maps, and helper functions instead of hardcoding colors or spacing.
- Lint with Stylelint when editing SCSS.

## Naming And File Hygiene

- Keep filenames and identifiers consistent with the surrounding subsystem.
- Prefer focused changes over opportunistic refactors.
- Avoid renaming public paths, generated content paths, or ATT&CK URL structures unless required.
- Preserve comments that explain generation behavior, browser compatibility, or build caveats.

## Agent Workflow Expectations

- Before editing, identify which subsystem you are in: root Python build, `attack-search/`, `attack-style/`, `attack-theme/`, or `modules/`.
- Run the narrowest relevant validation for the files you touched.
- If you changed `attack-search/`, run at least its relevant Jest or ESLint command.
- If you changed SCSS, run `cd attack-style && npm run lint` and usually rebuild styles.
- If you changed root build or content-generation code, run at least the relevant `update-attack.py` build or targeted test category.
- If your change affects rendered output or routing, validate with the Docker Nginx test environment when practical.

## Git And Contribution Notes

- Pull requests should target the `develop` branch per `CONTRIBUTING.md`.
- The PR template expects a reviewer and a `CHANGELOG.md` update when appropriate.
- Do not assume `master` is the integration branch just because GitHub Pages deploys from it.
- Use Conventional Commit style git messages

## When Unsure

- Read the nearest config file and a nearby edited file before making stylistic changes.
- Prefer matching existing conventions over introducing new tools or patterns.
- Keep builds reproducible, paths stable, and generated output compatible with the current pipeline.
