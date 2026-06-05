---
description: Generate a technical implementation plan from a multi-site spec and save it as a companion plan file
---

# speckit.spec-kit-multi-sites.plan

You are running the **Multi-Sites Plan** command. This command reads an existing spec file and generates a structured technical implementation plan, saved as a companion `*-plan.md` file next to the spec.

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
Compute `$SPEC_DIR` using the same rules as the specify command:

- If `$SPEC_MODE` is `targeted`:
  - If `$TARGET` is `core`: `$SPEC_DIR` = `specs/`
  - Otherwise: `$SPEC_DIR` = `$SITES_FOLDER/$TARGET/specs/`
- If `$SPEC_MODE` is `single`: `$SPEC_DIR` = `specs/`

---

## STEP 3 — Choose Spec File

List all available specs from `$SPEC_DIR`:

- **New structure:** Find all sub-directories inside `$SPEC_DIR` matching `NNN-*` (if `$SPEC_MODE` is `targeted`) or `NNN-<$TARGET>-*` (if `$SPEC_MODE` is `single`) that contain a `spec.md` file. Show the directory name as the option label.
- **Legacy structure:** Find all `.md` files directly in `$SPEC_DIR` matching `NNN-*.md` (if `$SPEC_MODE` is `targeted`) or `NNN-<$TARGET>-*.md` (if `$SPEC_MODE` is `single`) that do not end in `-plan.md`. Show the filename as the option label.

Sort all options alphabetically. If no matching directories or legacy files are found, stop and tell the user:

  > **No specs found** in `$SPEC_DIR`. Run `/speckit.spec-kit-multi-sites.specify` to create one first.

Use `vscode_askQuestions` to ask the user which spec to plan:

```
vscode_askQuestions({
  questions: [{
    header: "spec_file",
    question: "Which spec would you like to plan?",
    options: [
      { label: "<name>", description: "$SPEC_DIR/<name>" },
      // ... one entry per found spec (either the new directory name or the legacy filename)
    ],
    allowFreeformInput: false
  }]
})
```

Based on the selected option, check if it's a directory or a legacy file structure:

- **If a directory (new structure):**
  - Set `$FEATURE_DIR` = `$SPEC_DIR/<selected name>`
  - Set `$SPEC_PATH` = `$FEATURE_DIR/spec.md`
  - Parse the directory name `<selected name>` to extract:
    - `$SPEC_NUM` — the leading zero-padded number (e.g. `001` from `001-user-authentication`)
    - `$FEATURE_NAME` — the remainder after stripping the number prefix (and target prefix in single mode)
      - targeted: `001-user-authentication` → `$FEATURE_NAME` = `user-authentication`
      - single: `001-websiteA-user-authentication` → `$FEATURE_NAME` = `user-authentication`

- **If a legacy file (legacy structure):**
  - Set `$SPEC_PATH` = `$SPEC_DIR/<selected name>`
  - Parse the filename to extract:
    - `$SPEC_NUM` — the leading zero-padded number (e.g. `001` from `001-user-authentication.md`)
    - `$FEATURE_NAME` — the remainder after stripping the number prefix (and target prefix in single mode)
      - targeted: `001-user-authentication.md` → `$FEATURE_NAME` = `user-authentication`
      - single: `001-websiteA-user-authentication.md` → `$FEATURE_NAME` = `user-authentication`

Read the file contents at `$SPEC_PATH`.

---

## STEP 4 — Compute Plan File Path

Derive the plan file path:

- **If using the new directory structure:**
  - `$PLAN_PATH` = `$FEATURE_DIR/plan.md`

- **If using the legacy file structure:**
  - If `$SPEC_MODE` is `targeted`: `$PLAN_PATH` = `$SPEC_DIR/$SPEC_NUM-$FEATURE_NAME-plan.md`
  - If `$SPEC_MODE` is `single`: `$PLAN_PATH` = `$SPEC_DIR/$SPEC_NUM-$TARGET-$FEATURE_NAME-plan.md`

If a plan file already exists at `$PLAN_PATH`, ask:

```
vscode_askQuestions({
  questions: [{
    header: "overwrite",
    question: "A plan already exists for this spec. What would you like to do?",
    message: "Existing plan: `$PLAN_PATH`",
    options: [
      { label: "Regenerate plan", description: "Overwrite the existing plan", recommended: true },
      { label: "Cancel" }
    ],
    allowFreeformInput: false
  }]
})
```

If the user selects "Cancel", stop with a friendly message.

---

## STEP 5 — Validate the Spec

Before generating the plan, briefly check that the spec is usable:

- If the Overview, Goals, and Acceptance Criteria sections are all empty or placeholder-only, stop and tell the user:

  > **Spec is not ready for planning.** Run `/speckit.spec-kit-multi-sites.clarify` and `/speckit.spec-kit-multi-sites.checklist` first to fill in the required sections.

Otherwise proceed.

---

## STEP 6 — Generate the Plan

Based on the full spec content (Overview, Goals, Non-Goals, User Stories, Acceptance Criteria, Technical Notes), produce a technical implementation plan using the template below. Substitute all `{{ }}` placeholders with real content derived from the spec.

### Plan File Template

```markdown
# Implementation Plan: {{ $FEATURE_NAME | title-case }}

**Spec:** `{{ $SPEC_PATH }}`
**Target:** {{ $TARGET }}
**Spec mode:** {{ $SPEC_MODE }}
**Date:** {{ TODAY_DATE }}

---

## Summary

<!-- 2–3 sentence summary of what will be built and the key technical approach. -->

## Architecture

<!-- Describe the components, modules, or layers involved. Use bullet points or a short diagram. -->

- 

## Implementation Steps

<!-- Ordered list of concrete implementation steps. Each step should be completable in one sitting. -->

1. 
2. 
3. 

## File Changes

<!-- List the files expected to be created, modified, or deleted. -->

| Action | File | Reason |
|--------|------|--------|
| Create | | |

## Dependencies

<!-- External packages, services, APIs, or other specs that must be in place before starting. -->

- 

## Testing Strategy

<!-- How will this feature be tested? Unit tests, integration tests, manual QA, e2e? -->

- 

## Risks and Mitigations

<!-- Known risks and how to handle them. Leave blank if none identified. -->

- 

## Open Questions

<!-- Technical unknowns that need resolution before or during implementation. -->

- [ ] 
```

---

## STEP 7 — Save the Plan File

Create the file at `$PLAN_PATH`. If the parent directory of `$PLAN_PATH` does not exist, create it (including any intermediate directories).

---

## STEP 8 — Report

Print:

---

✅ **Plan created!**

- **Spec:** `$SPEC_PATH`
- **Plan:** `$PLAN_PATH`
- **Target:** `$TARGET`
- **Mode:** `$SPEC_MODE`

**Next steps:**
1. Open `$PLAN_PATH` and review the generated plan.
2. Fill in any sections marked with `<!-- ... -->` that need your input.
3. Run `/speckit.spec-kit-multi-sites.tasks` to generate a granular task list for development.
4. Implement the feature by working through the tasks.
