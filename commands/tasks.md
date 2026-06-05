---
description: Generate an implementation task list from a spec and plan, and save it as a companion tasks.md file
---

# speckit.spec-kit-multi-sites.tasks

You are running the **Multi-Sites Tasks** command. This command reads an existing spec and its implementation plan to generate a granular, actionable task list for developers, saved as a `tasks.md` file in the feature directory.

Follow every step below exactly and in order. Do not skip steps.

**Important:** Whenever you need input from the user, always use the `vscode_askQuestions` tool. Never ask the user to type answers directly in the chat.

---

## STEP 1 — Load Config

Read the file at:
`.specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.yml`

- If the file exists and contains a non-empty `sites_folder` value, set `$SITES_FOLDER` to that value.
- If the file exists and contains a non-empty `spec_mode` value, set `$SPEC_MODE` to that value.
- If either value is missing, blank, or the file does not exist, stop and tell the user:

  > **Config not found.** Run `/speckit.spec-kit-multi-sites.specify` first — it will create the project config automatically on the first run.

---

## STEP 2 — Choose Target

List the immediate sub-directories inside `$SITES_FOLDER`. Exclude any directory named `all`, `default`, `.git`, `node_modules`, or any file (not folder). Sort alphabetically.

Use `vscode_askQuestions` to ask the user which target the spec belongs to:

```
vscode_askQuestions({
  questions: [{
    header: "target",
    question: "Which site (or core) does the spec belong to?",
    options: [
      { label: "core", description: "Root-level specs/ folder" },
      { label: "<website-1>", description: "$SITES_FOLDER/<website-1>" },
      { label: "<website-2>", description: "$SITES_FOLDER/<website-2>" },
      // ... one entry per detected sub-directory
    ],
    allowFreeformInput: false
  }]
})
```

Store the answer as `$TARGET`.

Compute `$SPEC_DIR` using the same rules as the specify command:

- If `$SPEC_MODE` is `targeted`:
  - If `$TARGET` is `core`: `$SPEC_DIR` = `specs/`
  - Otherwise: `$SPEC_DIR` = `$SITES_FOLDER/$TARGET/specs/`
- If `$SPEC_MODE` is `single`: `$SPEC_DIR` = `specs/`

---

## STEP 3 — Choose Spec File

List all available specs from `$SPEC_DIR`:

- **New structure:** Find all sub-directories inside `$SPEC_DIR` matching `NNN-*` (if `$SPEC_MODE` is `targeted`) or `NNN-<$TARGET>-*` (if `$SPEC_MODE` is `single`) that contain a `spec.md` file. For these, the spec file path is `$SPEC_DIR/<directory-name>/spec.md`. Show the directory name as the option label.
- **Legacy structure:** Find all `.md` files directly in `$SPEC_DIR` matching `NNN-*.md` (if `$SPEC_MODE` is `targeted`) or `NNN-<$TARGET>-*` (if `$SPEC_MODE` is `single`) that do not end in `-plan.md`, `-research.md`, or `-tasks.md`. For these, the spec file path is `$SPEC_DIR/<filename>`. Show the filename as the option label.

Sort all options alphabetically. If no matching directories or legacy files are found, stop and tell the user:

  > **No specs found** in `$SPEC_DIR`. Run `/speckit.spec-kit-multi-sites.specify` to create one first.

Use `vscode_askQuestions` to ask the user which spec to generate tasks for:

```
vscode_askQuestions({
  questions: [{
    header: "spec_file",
    question: "Which spec would you like to generate tasks for?",
    options: [
      { label: "<name>", description: "$SPEC_DIR/<name>" },
      // ... one entry per found spec (either the new directory name or the legacy filename)
    ],
    allowFreeformInput: false
  }]
})
```

Based on the selected option:

- **If a directory (new structure):**
  - Set `$FEATURE_DIR` = `$SPEC_DIR/<selected name>`
  - Set `$SPEC_PATH` = `$FEATURE_DIR/spec.md`
  - Set `$PLAN_PATH` = `$FEATURE_DIR/plan.md`
  - Set `$TASKS_PATH` = `$FEATURE_DIR/tasks.md`
- **If a legacy file (legacy structure):**
  - Set `$SPEC_PATH` = `$SPEC_DIR/<selected name>`
  - Parse the filename to get `$BASE_NAME` (e.g. `001-feature.md` -> `001-feature`)
  - Set `$PLAN_PATH` = `$SPEC_DIR/$BASE_NAME-plan.md`
  - Set `$TASKS_PATH` = `$SPEC_DIR/$BASE_NAME-tasks.md`

Read the file contents at `$SPEC_PATH` and search for the plan at `$PLAN_PATH`. If the plan does not exist, stop and suggest running `/speckit.spec-kit-multi-sites.plan` first.

---

## STEP 4 — Compute Feature Info

Parse `$SPEC_PATH` or its content to get `$FEATURE_NAME`.

---

## STEP 5 — Generate Tasks File

Create the tasks file at `$TASKS_PATH` using the template below. The tasks should be derived from the "Implementation Steps" and "File Changes" sections of the plan, broken down into small, verifiable items.

### Tasks File Template

```markdown
# Implementation Tasks: {{ $FEATURE_NAME | title-case }}

**Spec:** `{{ $SPEC_PATH }}`
**Plan:** `{{ $PLAN_PATH }}`
**Target:** {{ $TARGET }}
**Date:** {{ TODAY_DATE }}

---

## Development Tasks

<!-- High-level development phases and their granular tasks. -->

### [ ] Phase 1: Setup
- [ ] 
- [ ] 

### [ ] Phase 2: Implementation
- [ ] 
- [ ] 

### [ ] Phase 3: Testing & Verification
- [ ] 
- [ ] 

## Documentation
- [ ] 

## Review & Deployment
- [ ] 
```

---

## STEP 6 — Save and Report

Create the file at `$TASKS_PATH`. Ensure the directory exists.

Print:

---

✅ **Tasks file created!**

- **Spec:** `$SPEC_PATH`
- **Tasks:** `$TASKS_PATH`
- **Target:** `$TARGET`

**Next steps:**
1. Open `$TASKS_PATH` and track your progress as you implement the feature.
2. Use the task list during code reviews to ensure all requirements are met.
