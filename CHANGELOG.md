# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.6.0] — 2026-06-05

### Added

- `speckit.spec-kit-multi-sites.research` command — conducts technical research and documentation for a spec.
- `speckit.spec-kit-multi-sites.tasks` command — generates a granular implementation task list from the spec and plan.

### Changed

- Updated all commands to suggest the next logical step in the Spec Kit workflow.
- Updated README to document the full 6-step workflow (`specify` -> `clarify` -> `checklist` -> `research` -> `plan` -> `tasks`).

## [1.5.0] — 2026-06-05

### Changed

- Updated spec organisation to use feature-specific directories instead of placing flat files directly in the `specs/` folder.
- Standardized filenames within feature directories to [spec.md](spec.md) and [plan.md](plan.md).
- Maintained backward compatibility for legacy flat-file specs in all commands (`specify`, `clarify`, `checklist`, `plan`).
- Updated README to reflect the new directory structure and clean naming conventions.

## [1.4.0] — 2026-06-02

### Added

- `speckit.spec-kit-multi-sites.clarify` command — reviews a spec file, identifies empty sections, vague acceptance criteria, incomplete user stories, and unresolved open questions; collects answers from the user via quick-pick and updates the file in place
- `speckit.spec-kit-multi-sites.checklist` command — evaluates a spec against an 8-point readiness checklist (Overview, Goals, Non-Goals, User Stories, Acceptance Criteria, Open Questions, Target, Technical Notes) and reports pass/partial/fail per criterion
- `speckit.spec-kit-multi-sites.plan` command — generates a structured technical implementation plan (Summary, Architecture, Implementation Steps, File Changes, Dependencies, Testing Strategy, Risks, Open Questions) saved as a companion `*-plan.md` file next to the spec

### Changed

- `commands/specify.md` Step 8 next-steps now references the three new multi-site-aware commands (`clarify`, `checklist`, `plan`) instead of the generic `speckit.clarify` / `speckit.plan`
- README Workflow Integration section expanded with a command reference table explaining each command's purpose

## [1.3.1] — 2026-06-02

### Fixed

- Removed invalid `mode: agent` frontmatter from `commands/specify.md`; SpecKit command files only accept `description`, `tools`, and `scripts` in their frontmatter — the spurious field introduced in v1.1.0 caused the command to fail to register or execute correctly

## [1.3.0] — 2026-06-02

### Changed

- Expanded Usage section in README with step-by-step examples and quick-pick UI illustrations for all 6 steps

## [1.2.0] — 2026-06-02

### Changed

- Sites folder and spec mode are now asked only once per project; answers are persisted to `spec-kit-multi-sites-config.yml` and reused on subsequent runs

## [1.1.0] — 2026-06-02

### Fixed

- Command name now uses correct namespace: `speckit.spec-kit-multi-sites.specify`
- Added required `mode: agent` YAML frontmatter to command file for agent recognition
- Updated all documentation to reference correct command name

## [1.0.0] — 2026-05-29

### Added

- Initial release of the Multi-Sites Spec Kit extension.
- `speckit.spec-kit-multi-sites.specify` command with interactive multi-step flow.
- Auto-detection of sites folder for Drupal (`docroot/sites`, `web/sites`, `sites/`) and generic multi-site layouts.
- **Targeted specs mode**: per-site `specs/` folders with independent auto-increment counters.
- **Single specs mode**: shared root `specs/` folder with site-scoped counters embedded in file names.
- `core` target option for root-level shared feature specs.
- Standard spec file template (Overview, Goals, Non-Goals, User Stories, Acceptance Criteria, Technical Notes, Open Questions, References).
- Configuration template (`config-template.yml`) for locking in `sites_folder`, `spec_mode`, and `excluded_sites`.
- `.gitignore` with sensible defaults.
