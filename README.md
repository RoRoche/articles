# Articles

This repository contains long-form technical articles and the tooling used to
validate them.

## Project layout

```text
.
├── .github/
│   ├── dependabot.yml       # Weekly dependency update checks
│   └── workflows/
│       └── lint.yml         # Lint checks on pushes and pull requests
├── rethinking-hexagonal-architecture-with-elegant-objects/
│   ├── header.png           # Article header image
│   └── post.md              # Article source
├── .pre-commit-config.yaml  # Local pre-commit hooks
├── .textlintrc.json         # Prose lint configuration
├── .markdownlint-cli2.jsonc # Markdown lint configuration
├── LICENSE                  # MIT license
├── package.json             # Validation scripts and tool dependencies
└── package-lock.json        # Locked npm dependency versions
```

Each article lives in its own directory, with its Markdown source and any
associated assets together.

## Validate articles

Install the JavaScript tooling:

```sh
npm ci
```

Run both validators:

```sh
npm run lint
```

Or run them separately:

```sh
npm run lint:prose
npm run lint:markdown
```

The pre-commit configuration runs both validators when Markdown files change.
To enable the hooks in a Git clone, install [pre-commit](https://pre-commit.com/)
and run:

```sh
pre-commit install
```

GitHub Actions runs the same npm lint scripts on pushes and pull requests.
Dependabot checks npm dependencies and GitHub Actions for updates weekly.
