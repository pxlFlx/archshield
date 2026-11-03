# ArchShield

Local-first architecture checks for TypeScript and JavaScript repositories. Define module boundaries in `archshield.json`, run `archshield check`, and get file-level findings locally or in CI.

## Quick start

Requirements: Node.js 20.19 or newer.

```sh
cd archshield
npm ci
npm run build
npm link
cd ../your-project
archshield init --preset clean-architecture
archshield check
```

`init` creates a starter configuration and never overwrites an existing file unless `--force` is passed. To use the React preset, run `archshield init --preset react`.

When ArchShield is published to npm, add it to a target repository with `npm install --save-dev archshield` and invoke it with `npx archshield check`.

## Example

```text
ERROR  no-presentation-to-infrastructure
src/components/UserPanel.tsx:12:1
Presentation code must not depend directly on infrastructure.
Why it matters: A direct dependency couples presentation to persistence details.
Suggested fix: Move the call behind an application-level service.
```

The example is representative of the text reporter; exact file names and counts depend on the scanned project.

## Configure rules

Create `archshield.json` in the project root, or use `archshield init`. Configuration format version 1 is validated before scanning. See [`archshield.example.json`](archshield.example.json), the [configuration guide](docs/configuration.md), and the machine-readable [JSON Schema](docs/archshield.schema.json).

```json
{
  "version": 1,
  "include": ["src/**/*.{ts,tsx,js,jsx,mjs,cjs}"],
  "layers": [
    {
      "name": "presentation",
      "patterns": ["src/components/**", "src/pages/**"]
    },
    {
      "name": "infrastructure",
      "patterns": ["src/database/**", "src/repositories/**"]
    }
  ],
  "rules": [
    {
      "id": "no-ui-to-database",
      "type": "layer-boundary",
      "severity": "error",
      "from": ["presentation"],
      "to": ["infrastructure"]
    }
  ]
}
```

Supported rule types are `layer-boundary`, `forbidden-import`, `allowed-dependency`, and `no-cycles`. A rule's `severity` is `error`, `warning`, or `info`. See [rules and matching](docs/rules.md).

## Commands

```sh
archshield init [--preset clean-architecture|react] [--force]
archshield check [--config path] [--format text|json|sarif] [--output path]
archshield check [--baseline path] [--update-baseline] [--strict-baseline]
archshield check [--changed-only --base origin/main]
archshield graph [--format mermaid|dot] [--include-external] [--output path]
```

Exit codes: `0` means no blocking findings; `1` means blocking findings; `2` means invalid invocation/configuration or a fatal analysis error. Warnings do not fail by default; use `--fail-on-warn` to make new warnings blocking. Output is sorted and uses project-relative paths. JSON reports use schema version 1 and include diagnostics, dependency counts, summary counts, findings, and stable fingerprints.

## Baselines and changed files

Save a reviewed snapshot with `archshield check --update-baseline`. By default this writes `.archshield-baseline.json`; pass `--baseline path` to choose another location. Existing baseline findings do not fail by default. New findings still follow the configured severity policy. `--strict-baseline` makes existing findings blocking too. Review baseline changes carefully: updating a baseline can hide newly introduced violations.

`--changed-only` requires `--base <ref>`. ArchShield analyzes the full project, then displays findings whose importer changed. Fetch the base ref and enough history before the check; shallow clones may not contain the required merge base. Git is invoked with argument arrays, not a shell.

## GitHub Actions

Install ArchShield as a development dependency in the repository before using the example workflow. [`archshield-example.yml`](.github/workflows/archshield-example.yml) uses pull-request and push events with read-only repository permissions. Findings become workflow annotations automatically in GitHub Actions. The repository's own build and tests run in [`ci.yml`](.github/workflows/ci.yml). See [GitHub Actions setup](docs/github-action.md).

SARIF can be written locally using `archshield check --format sarif --output archshield.sarif`. Uploading SARIF is not included in the example workflow and requires the narrowly scoped `security-events: write` permission.

## Scope and security

- Full import extraction is implemented and tested for TypeScript and JavaScript: static imports, re-exports, literal dynamic imports, and literal CommonJS `require`.
- PHP and Python adapters are not implemented. Enabling either fails with a clear configuration error; ArchShield does not claim support for them.
- Local import resolution uses TypeScript's compiler API and honors `tsconfig` path aliases when resolvable. External dependencies are not installed or executed.
- The scanner never runs project code, installs project dependencies, downloads remote configuration, or sends source code to a service.
- Symlinks are not followed by default; source file size is limited; `.gitignore` and configured exclusions are respected.
- Inline suppressions, remote dashboards, and PR review comments are planned, not implemented.

## Development

```sh
npm ci
npm run build
npm run typecheck
npm run lint
npm run format:check
npm test
```

`npm test` builds the CLI and runs unit and temporary-project integration tests without network access. No performance benchmarks are published; scan time depends on the repository and its configuration.

## Examples

- [`examples/clean-architecture`](examples/clean-architecture) demonstrates an allowed dependency direction.
- [`examples/violating`](examples/violating) intentionally contains a boundary violation.

See [architecture](docs/architecture.md) and [troubleshooting](docs/troubleshooting.md) for implementation notes and common errors.
