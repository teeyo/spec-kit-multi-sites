# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

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
