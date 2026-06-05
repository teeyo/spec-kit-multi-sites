---
description: Review a multi-site spec and resolve ambiguities, gaps, and open questions with the user
---

# speckit.spec-kit-multi-sites.clarify

You are running the **Multi-Sites Clarify** command. This command reviews an existing spec file, identifies gaps and ambiguities, collects the user's answers, and updates the spec in place.

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

Use `vscode_askQuestions` to ask the user which spec to clarify:

```
vscode_askQuestions({
  questions: [{
    header: "spec_file",
    question: "Which spec would you like to clarify?",
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

## STEP 4 — Identify Gaps

Review the spec content and produce an internal list of issues across these categories:

1. **Empty or placeholder sections** — sections that contain only an HTML comment (`<!-- ... -->`) or a bare `-` with no text after it.
2. **Vague acceptance criteria** — checkbox items that are not concretely testable (e.g., "works well", "is fast", "is user-friendly", "looks good").
3. **Unresolved open questions** — items in the "Open Questions" section that are not crossed out and have no resolution note.
4. **Incomplete user stories** — stories missing a role, an action, or a benefit in the "As a / I want / so that" structure.

If **no issues** are found across all categories, skip Steps 5 and 6 and go directly to Step 7, reporting that the spec looks complete.

---

## STEP 5 — Collect Answers

For each issue identified, use `vscode_askQuestions` to ask one focused question. Show the relevant excerpt from the spec as the `message` field so the user has context.

Group closely related questions in a single `vscode_askQuestions` call when they share context. Do not ask more than three questions in a single call.

Examples:

```
// For an empty Goals section:
vscode_askQuestions({
  questions: [{
    header: "goals",
    question: "What are the main goals this feature should achieve?",
    message: "The Goals section is currently empty. List goals separated by line breaks."
  }]
})

// For a vague acceptance criterion:
vscode_askQuestions({
  questions: [{
    header: "ac_1",
    question: "How would you make this acceptance criterion testable?",
    message: "Current criterion: `- [ ] The feature works well`\n\nDescribe a concrete, verifiable condition."
  }]
})
```

Collect all answers before modifying the file.

---

## STEP 6 — Update the Spec File

Apply all collected answers to the spec file:

- Fill in empty sections with the user's text, formatted as bullet points where appropriate.
- Replace vague acceptance criteria with specific, testable conditions using the user's clarification.
- Mark resolved open questions by striking them out with `~~text~~` and appending a resolution note inline, **or** by removing them from Open Questions and moving the answer to the Technical Notes or Acceptance Criteria section where it belongs.
- Fill in incomplete user stories using the user's answers.

Rewrite the file in place. Preserve all original structure, headings, and content that was not part of an identified issue.

---

## STEP 7 — Report

After updating the file (or if no issues were found), print:

---

✅ **Spec clarified!**

- **File:** `$SPEC_PATH`
- **Updates made:** [list each section that was changed, or "No changes needed — spec was already complete"]

**Next step:** Run `/speckit.spec-kit-multi-sites.checklist` to verify the spec is implementation-ready.
