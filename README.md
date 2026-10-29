## Backend module structure

Architecture decisiones are documented at:
[ADRs](docs/adr/README.md).

The backend is a modular monolith. Each business module is compiled as one project;
its internal `Domain`, `Application`, `Infrastructure`, and `Contracts` folders are
organizational boundaries, not separate projects.

```text
backend/src/
  Api/                       HTTP host and composition root
  Abstractions/              Minimal shared technical contracts
  Modules/
    Catalog/                 F1 reference data
    RaceData/                Observed racing data from external sources
    Statistics/              Descriptive metrics calculated from race data
    Comparison/              Normalized driver and team comparisons
    Performance/             Ratings derived from comparisons and statistics
```

Module references are intentionally one-way:

```text
Catalog <- RaceData <- Statistics <- Comparison <- Performance
```

`Api` may reference every module to expose HTTP endpoints and compose the
application. Modules must never reference `Api` or a module to their right
in the dependency chain.

## Backend coding standards and naming conventions


| Object Name               | Notation   | Length | Plural | Prefix | Suffix | Abbreviation | Char Mask          | Underscores |
|:--------------------------|:-----------|-------:|:-------|:-------|:-------|:-------------|:-------------------|:------------|
| Namespace name            | PascalCase |    128 | Yes    | Yes    | No     | No           | [A-z][0-9]         | No          |
| Class name                | PascalCase |    128 | No     | No     | Yes    | No           | [A-z][0-9]         | No          |
| Constructor name          | PascalCase |    128 | No     | No     | Yes    | No           | [A-z][0-9]         | No          |
| Method name               | PascalCase |    128 | Yes    | No     | No     | No           | [A-z][0-9]         | No          |
| Method arguments          | camelCase  |    128 | Yes    | No     | No     | Yes          | [A-z][0-9]         | No          |
| Local variables           | camelCase  |     50 | Yes    | No     | No     | Yes          | [A-z][0-9]         | No          |
| Constants name            | PascalCase |     50 | No     | No     | No     | No           | [A-z][0-9]         | No          |
| Field name Public         | PascalCase |     50 | Yes    | No     | No     | Yes          | [A-z][0-9]         | No          |
| Field name Private        | _camelCase |     50 | Yes    | No     | No     | Yes          | _[A-z][0-9]        | Yes         |
| Properties name           | PascalCase |     50 | Yes    | No     | No     | Yes          | [A-z][0-9]         | No          |
| Delegate name             | PascalCase |    128 | No     | No     | Yes    | Yes          | [A-z]              | No          |
| Enum type name            | PascalCase |    128 | Yes    | No     | No     | No           | [A-z]              | No          |

## Branching conventions

<type>/<description>
Purpose Prefixes — describe the intent of the work:

feature/ (or feat/): For new features (e.g., feature/add-login-page, feat/add-login-page)
bugfix/ (or fix/): For bug fixes (e.g., bugfix/fix-header-bug, fix/header-bug)
hotfix/: For urgent fixes (e.g., hotfix/security-patch)
release/: For branches preparing a release (e.g., release/v1.2.0)
chore/: For non-code tasks like dependency, docs updates (e.g., chore/update-dependencies)
