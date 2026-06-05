---
description: Verify a multi-site spec is complete and implementation-ready against a structured checklist
---

# speckit.spec-kit-multi-sites.checklist

You are running the **Multi-Sites Checklist** command. This command evaluates an existing spec file against a structured readiness checklist and tells you what is complete, what is partial, and what is still missing.

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
- **Legacy structure:** Find all `.md` files directly in `$SPEC_DIR` matching `NNN-*.md` (if `$SPEC_MODE` is `targeted`) or `NNN-<$TARGET>-*.md` (if `$SPEC_MODE` is `single`) that do not end in `-plan.md`. For these, the spec file path is `$SPEC_DIR/<filename>`. Show the filename as the option label.

Sort all options alphabetically. If no matching directories or legacy files are found, stop and tell the user:

  > **No specs found** in `$SPEC_DIR`. Run `/speckit.spec-kit-multi-sites.specify` to create one first.

Use `vscode_askQuestions` to ask the user which spec to check:

```
vscode_askQuestions({
  questions: [{
    header: "spec_file",
    question: "Which spec would you like to check?",
    options: [
      { label: "<name>", description: "$SPEC_DIR/<name>" },
      // ... one entry per found spec (either the new directory name or the legacy filename)
    ],
    allowFreeformInput: false
  }]
})
```

Set `$SPEC_PATH` to the full path of the selected spec file (either `$SPEC_DIR/<selected directory>/spec.md` or `$SPEC_DIR/<selected filename>`). Read the file contents.

---

## STEP 4 — Run the Readiness Checklist

Evaluate the spec against each criterion below. Assign a status to each:

- ✅ **Pass** — the criterion is fully met
- ⚠️ **Partial** — the section exists but is thin or incomplete
- ❌ **Fail** — the section is absent or contains only placeholder content

| # | Criterion | Status |
|---|-----------|--------|
| 1 | **Overview** is present with at least 2 sentences describing what the feature does and why it is needed | |
| 2 | **Goals** has at least one concrete, specific goal (not a placeholder) | |
| 3 | **Non-Goals** has at least one explicit boundary | |
| 4 | **User Stories** — at least one story in "As a [role] / I want [action] / so that [benefit]" format with all three parts filled | |
| 5 | **Acceptance Criteria** — at least two checkbox items that are concrete and testable | |
| 6 | **Open Questions** — section is either empty or every item is marked resolved (struck through or removed) | |
| 7 | **Target** — the `**Target:**` line in the file matches `$TARGET` | |
| 8 | **Technical Notes** — section is present (content is optional for simple features, but the section heading must exist) | |

---

## STEP 5 — Report and Recommend

Print the completed checklist table.

**If all criteria are ✅ Pass**, print:

---

✅ **Spec is implementation-ready!**

- **File:** `$SPEC_PATH`

**Next step:** Run `/speckit.spec-kit-multi-sites.plan` to generate the technical implementation plan.

---

**If any criteria are ⚠️ Partial or ❌ Fail**, print:

---

⚠️ **Spec needs attention before planning.**

The following items require action:

- [for each failing or partial criterion: state the item number, the criterion name, and a one-sentence note on what is missing]

**Suggested action:** Run `/speckit.spec-kit-multi-sites.clarify` to resolve the issues above, then re-run this checklist.
