---
description: Perform technical research for a spec and save it as a companion research.md file
---

# speckit.spec-kit-multi-sites.research

You are running the **Multi-Sites Research** command. This command helps you explore technical unknowns, document findings, and make recommendations for a specific feature, saving the results as a `research.md` file in the feature directory.

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

Use `vscode_askQuestions` to ask the user which spec to research:

```
vscode_askQuestions({
  questions: [{
    header: "spec_file",
    question: "Which spec would you like to research?",
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
  - Set `$RESEARCH_PATH` = `$FEATURE_DIR/research.md`
- **If a legacy file (legacy structure):**
  - Set `$SPEC_PATH` = `$SPEC_DIR/<selected name>`
  - Parse the filename to get `$BASE_NAME` (e.g. `001-feature.md` -> `001-feature`)
  - Set `$RESEARCH_PATH` = `$SPEC_DIR/$BASE_NAME-research.md`

Read the file contents at `$SPEC_PATH`.

---

## STEP 4 — Compute Feature Info

Parse `$SPEC_PATH` or its content to get `$FEATURE_NAME`.

---

## STEP 5 — Perform Research

Explore the codebase and external documentation to address technical unknowns related to the spec. Document your findings.

---

## STEP 6 — Generate Research File

Create the research file at `$RESEARCH_PATH` using the template below.

### Research File Template

```markdown
# Research: {{ $FEATURE_NAME | title-case }}

**Spec:** `{{ $SPEC_PATH }}`
**Target:** {{ $TARGET }}
**Date:** {{ TODAY_DATE }}

---

## Context and Goals

<!-- Briefly explain the technical problem being researched. -->

## Methodology

<!-- How was this research conducted? (e.g., Codebase audit, proof of concept, API documentation review). -->

## Findings

<!-- Document what was discovered. Use bullet points and code snippets where helpful. -->

## Recommendations

<!-- What is the recommended technical approach based on the findings? -->

## Open Questions

<!-- Any technical unknowns that still remain. -->

- [ ] 
```

---

## STEP 7 — Save and Report

Create the file at `$RESEARCH_PATH`. Ensure the directory exists.

Print:

---

✅ **Research file created!**

- **Spec:** `$SPEC_PATH`
- **Research:** `$RESEARCH_PATH`
- **Target:** `$TARGET`

**Next step:** Run `/speckit.spec-kit-multi-sites.plan` to generate the technical implementation plan using these findings.
