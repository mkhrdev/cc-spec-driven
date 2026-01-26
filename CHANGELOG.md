# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-01-26

### Added
- **Hook enforcement mechanism**: Ensure critical checklist steps are not skipped
  - SessionStart hooks for environment validation (check project.yaml)
  - PostToolUse hooks for path validation and index sync reminders
  - Stop hooks for completion validation

### ⚠️ BREAKING CHANGES
- **Removed `products/` directory layer**
  - **Migration required**: Move contents from `products/{product}/` up to the product root directory
  - Commands must now run inside the product directory (where `project.yaml` is located)
  - `/dd-init` initializes current directory instead of creating `products/{name}/` subdirectory
  - `/dd-status` no longer scans for multiple products; shows current product only
  - `/dd-check` no longer accepts `[product]` parameter

### Changed
- **Architecture simplification**: Direct product directory structure
  - Simpler path: `features/`, `changes/`, `specs/`, `cases/` directly in root
  - Environment detection now checks for `project.yaml` instead of `products/`
- Updated plugin.json to include hooks configuration

### Documentation
- Updated directory structure in README and CLAUDE.md
- Added Hook benefits section to README

## [0.1.0] - 2025-01-10

### Added
- Initial release of cc-spec-driven plugin
- Core commands: `/dd-init`, `/dd-status`, `/dd-update`, `/dd-confirm`, `/dd-done`, `/dd-drop`
- Auxiliary commands: `/dd-check`, `/dd-rebase`, `/dd-spec-dev`, `/dd-spec-test`, `/dd-test-case`
- Execution environment check: prompts user when `products/` folder not found
- Bilingual support with `public/` folder structure

### Documentation
- Complete user guide in README
- Chinese translation available at `docs/translate/README.zh.md`
