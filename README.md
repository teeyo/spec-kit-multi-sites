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

### Known: installer creates symlinks

The `specify extension add` command installs extensions using `ln -s` (symlinks) rather than copying files. This means the extension resolves at runtime through the symlink, which is normally fine — but if the global Spec Kit store is cleaned up, updated, or moved, the symlink may break silently.

This behaviour repeats on every update or reinstall. If you find commands fail to load after an update, replace the symlink with a real copy:

```bash
# Run from the project root
EXT_DIR=".specify/extensions/spec-kit-multi-sites"
cp -rL "$EXT_DIR" "${EXT_DIR}-copy"
rm "$EXT_DIR"
mv "${EXT_DIR}-copy" "$EXT_DIR"
```

> **Tip:** If you want real files from the start, install from a local checkout with `--dev` (see above) and then copy the folder manually — the `--dev` flag already resolves to your local path, so a plain `cp -r` of the checkout is sufficient.

## Usage

In your AI coding agent's chat, run:

```
/speckit.spec-kit-multi-sites.specify
```

The command walks you through an interactive flow using VS Code quick-pick prompts. Steps 1 and 2 are only asked **once** — the answers are saved to your project config and reused on every subsequent run.

---

### Step 1 — Sites folder *(first run only)*

A quick-pick shows the auto-detected folder. Confirm it or choose "Enter manually" to type a custom path.

```
Which folder contains your website sub-folders?
❯ docroot/sites   (Auto-detected)
  Enter manually
```

> On a Drupal project the command detects `docroot/sites` automatically. On a generic multi-site setup it may detect `sites/`, `apps/`, `tenants/`, etc.

---

### Step 2 — Spec mode *(first run only)*

Choose how specs are organised across sites.

```
How would you like to organise your specs?
❯ Targeted specs   Each site has its own specs/ folder with independent numbering
  Single specs     All specs share a root specs/ folder; site name is in the file name
```

Both choices are persisted to `.specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.yml` so you are never asked again.

---

### Step 3 — Target site

Pick the site (or `core`) this spec belongs to. The list is built from sub-directories found inside your sites folder.

```
Where should this spec be created?
❯ core            Shared core feature — top-level, applies to all websites
  website-alpha   docroot/sites/website-alpha
  website-beta    docroot/sites/website-beta
  website-gamma   docroot/sites/website-gamma
```

---

### Step 4 — Feature name

Type a short description of the feature. Spaces are converted to hyphens automatically.

```
What is the name of this feature?
> user authentication
```

> Normalised to `user-authentication`.

---

### Step 5 — Confirmation

Review the computed path and branch name before anything is written to disk.

```
Ready to create the spec — confirm the details below:
Mode: targeted | Target: website-alpha | File: docroot/sites/website-alpha/specs/001-user-authentication.md | Branch: 001-user-authentication

❯ Create spec
  Cancel
```

---

### Step 6 — Spec file created

The spec file is created at the computed path using the standard Spec Kit template.

```
✅ Spec created successfully!

File:   docroot/sites/website-alpha/specs/001-user-authentication.md
Target: website-alpha
Mode:   targeted

Next steps:
1. Open the file and fill in the spec details.
2. Run /speckit.clarify to resolve ambiguities.
3. Run /speckit.plan to generate the technical plan.
```

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

All workflow commands are multi-site-aware — they read the saved config, ask which site (or core) you are working on, and scope all file operations to the correct `specs/` folder for that target.

```
/speckit.spec-kit-multi-sites.specify
  → /speckit.spec-kit-multi-sites.clarify
  → /speckit.spec-kit-multi-sites.checklist
  → /speckit.spec-kit-multi-sites.plan
```

| Command | Purpose |
|---------|--------|
| `speckit.spec-kit-multi-sites.specify` | Create a new spec in the correct site or core folder with auto-incremented numbering |
| `speckit.spec-kit-multi-sites.clarify` | Review a spec and resolve ambiguities, gaps, and open questions interactively |
| `speckit.spec-kit-multi-sites.checklist` | Evaluate a spec against a readiness checklist before planning |
| `speckit.spec-kit-multi-sites.plan` | Generate a technical implementation plan and save it as a companion `*-plan.md` file |

Every command after `specify` starts by loading the saved config (`sites_folder`, `spec_mode`), asking which target to work on, and listing the available spec files for that target — so you never have to re-configure the project and always work in the right scope.

## License

MIT — see [LICENSE](LICENSE)
