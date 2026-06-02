# Multi-Sites Spec Kit

A [Spec Kit](https://github.github.io/spec-kit/) community extension that brings multi-site awareness to the `/specify` workflow.

## Overview

In multi-site projects (Drupal, custom CMS, monorepos with multiple web apps) each website is an independent scope. This extension replaces the generic `/speckit.specify` step with a guided, site-aware flow that:

1. **Auto-detects** the sites folder from your project structure (e.g. `docroot/sites` for Drupal).
2. **Asks which site** (or `core`) the spec belongs to, listing all detected websites.
3. **Keeps auto-increment numbers separate per scope**, so each site (and core) starts at `001`.
4. **Supports two spec organisation modes** with different file placement and naming strategies.

## Spec Modes

### Targeted Specs

Each website has its own `specs/` folder inside its directory. Core specs go in a root-level `specs/` folder.

```
specs/                          ← core specs (001-feature-a.md, 002-feature-b.md)
docroot/sites/
  websiteA/
    specs/                      ← websiteA specs (001-login.md, 002-checkout.md)
  websiteB/
    specs/                      ← websiteB specs (001-homepage.md)
```

Auto-increment counters are fully independent: adding a spec to `websiteB` never affects `websiteA` or `core` counters.

### Single Specs

All specs share one root-level `specs/` folder. The site name is part of the file name, making each scope's counter independent within that shared folder.

```
specs/
  001-core-auth.md
  001-websiteA-login.md
  001-websiteB-homepage.md
  002-core-navigation.md
  002-websiteA-checkout.md
```

## Installation

```bash
specify extension add spec-kit-multi-sites
```

Or install from a local checkout during development:

```bash
specify extension add --dev /path/to/spec-kit-multi-sites
```

## Usage

In your AI coding agent's chat, run:

```
/speckit.spec-kit-multi-sites.specify
```

The command walks you through an interactive flow:

1. **Sites folder** — confirms or lets you override the detected folder.
2. **Spec mode** — choose `targeted` or `single`.
3. **Target** — pick `core` or one of the detected website names.
4. **Feature name** — describe what you are building.
5. **Confirmation** — preview the path and branch name before anything is written.
6. **Spec file created** — a standard spec template is placed in the correct location.

## Folder Detection

The extension scans the project root and applies these rules in order:

| Pattern | Framework |
|---------|-----------|
| `docroot/sites/` | Drupal (composer-based) |
| `web/sites/` | Drupal (alternative layout) |
| `sites/` | Drupal (legacy), generic CMS |
| `webs/`, `domains/`, `tenants/`, `apps/`, `vhosts/` | Generic multi-site setups |

If no sites folder is detected automatically, you will be prompted to enter the path manually.

## Configuration

Copy the template to create your project config:

```bash
cp .specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.template.yml \
   .specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.yml
```

Available options:

| Key | Default | Description |
|-----|---------|-------------|
| `sites_folder` | *(auto-detect)* | Override the sites folder path |
| `spec_mode` | *(ask every time)* | Lock in `targeted` or `single` |
| `excluded_sites` | `["all", "default"]` | Site sub-directories to ignore |

## Spec Template

Every generated spec follows the standard Spec Kit structure:

- **Overview** — what the feature does and why it exists
- **Goals / Non-Goals** — scope boundaries
- **User Stories** — role-based requirements
- **Acceptance Criteria** — testable checkboxes
- **Technical Notes** — constraints and architectural decisions
- **Open Questions** — items to resolve before implementation
- **References** — links to related specs, tickets, designs

## Workflow Integration

After creating a spec with this extension, continue the standard Spec Kit workflow:

```
/speckit.spec-kit-multi-sites.specify  →  /speckit.clarify  →  /speckit.checklist
  →  /speckit.plan  →  /speckit.tasks  →  /speckit.analyze  →  /speckit.implement
```

## License

MIT — see [LICENSE](LICENSE)
