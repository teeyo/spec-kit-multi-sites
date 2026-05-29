# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-05-29

### Added

- Initial release of the Multi-Sites Spec Kit extension.
- `speckit.multisite.specify` command with interactive multi-step flow.
- Auto-detection of sites folder for Drupal (`docroot/sites`, `web/sites`, `sites/`) and generic multi-site layouts.
- **Targeted specs mode**: per-site `specs/` folders with independent auto-increment counters.
- **Single specs mode**: shared root `specs/` folder with site-scoped counters embedded in file names.
- `core` target option for root-level shared feature specs.
- Standard spec file template (Overview, Goals, Non-Goals, User Stories, Acceptance Criteria, Technical Notes, Open Questions, References).
- Configuration template (`config-template.yml`) for locking in `sites_folder`, `spec_mode`, and `excluded_sites`.
- `.gitignore` with sensible defaults.
