---
description: Create a spec in the right site or core folder with per-site auto-incremented numbering
---

# speckit.spec-kit-multi-sites.specify

You are running the **Multi-Sites Specify** command. This is a multi-site-aware replacement for `/speckit.specify`. Follow every step below exactly and in order. Do not skip steps. Do not proceed to a later step until the previous step is complete.

**Important:** Whenever you need input from the user, always use the `vscode_askQuestions` tool to present a VS Code quick-pick or input box UI. Never ask the user to type answers directly in the chat.

---

## STEP 1 — Load or Detect the Sites Folder

**First**, check whether the project already has a saved config by reading the file at:
`.specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.yml`

- If the file exists **and** contains a non-empty `sites_folder` value, set `$SITES_FOLDER` to that value and **skip the rest of this step** — do not ask the user anything.
- Otherwise, continue below.

Scan the project's directory structure to find the folder that contains the individual website sub-folders.

Apply the following detection rules, in priority order:

1. **Drupal multi-site** — look for `docroot/sites/` or `web/sites/` or `sites/`. A Drupal sites folder contains sub-directories that each have a `settings.php` or `settings.local.php` file inside them.
2. **Generic CMS / framework** — look for any folder named `sites/`, `webs/`, `domains/`, `tenants/`, `apps/`, or `vhosts/` at the root or one level deep that contains at least two sub-directories.
3. **Custom** — if no candidate is found automatically, set `detected_sites_folder` to `null`.

Use `vscode_askQuestions` to confirm the detected folder with the user. Build the options list as follows:

- If a folder was detected, include it as the first option marked as recommended.
- Always include an "Enter manually" option at the end.

```
vscode_askQuestions({
  questions: [{
    header: "sites_folder",
    question: "Which folder contains your website sub-folders?",
    options: [
      { label: "<detected_sites_folder>", description: "Auto-detected", recommended: true },
      { label: "Enter manually", description: "Type a custom path" }
    ]
  }]
})
```

- If the user selects "Enter manually", call `vscode_askQuestions` again with a free-text question (no options) to collect the path.
- If no folder was detected, call `vscode_askQuestions` directly with a free-text question asking for the path.

Store the confirmed value as `$SITES_FOLDER`.

**After confirming**, persist the value so this question is never asked again for this project:
- If `.specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.yml` does not exist, create it with the content:
  ```yaml
  sites_folder: "$SITES_FOLDER"
  ```
- If it already exists but `sites_folder` is blank or commented out, update that line to set `sites_folder: "$SITES_FOLDER"`.

---

## STEP 2 — Load or Choose the Spec Mode

**First**, check `.specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.yml`:

- If the file exists **and** contains a non-empty `spec_mode` value of either `targeted` or `single`, set `$SPEC_MODE` to that value and **skip the rest of this step** — do not ask the user anything.
- Otherwise, continue below.

Use `vscode_askQuestions` to ask the user to choose the spec organisation mode:

```
vscode_askQuestions({
  questions: [{
    header: "spec_mode",
    question: "How would you like to organise your specs?",
    options: [
      {
        label: "Targeted specs",
        description: "Each site has its own specs/ folder with independent numbering (recommended for large multi-site projects)",
        recommended: true
      },
      {
        label: "Single specs",
        description: "All specs share a root specs/ folder; site name is embedded in the file name"
      }
    ],
    allowFreeformInput: false
  }]
})
```

Map the selection: "Targeted specs" → `targeted`, "Single specs" → `single`.

Store the choice as `$SPEC_MODE`.

**After confirming**, persist the value to `.specify/extensions/spec-kit-multi-sites/spec-kit-multi-sites-config.yml`:
- If `spec_mode` is already a line in the file (even commented out), update it to `spec_mode: "$SPEC_MODE"`.
- If the line does not exist, append `spec_mode: "$SPEC_MODE"` to the file.

---

## STEP 3 — Choose the Target (Website or Core)

List the immediate sub-directories inside `$SITES_FOLDER`. Exclude any directory named `all`, `default`, `.git`, `node_modules`, or any file (not folder). Sort alphabetically.

Use `vscode_askQuestions` to present the target list. Build the options dynamically:

```
vscode_askQuestions({
  questions: [{
    header: "target",
    question: "Where should this spec be created?",
    options: [
      { label: "core", description: "Shared core feature — top-level, applies to all websites" },
      { label: "<website-1>", description: "$SITES_FOLDER/<website-1>" },
      { label: "<website-2>", description: "$SITES_FOLDER/<website-2>" },
      // ... one entry per detected sub-directory
    ],
    allowFreeformInput: false
  }]
})
```

Store the answer as `$TARGET`.

---

## STEP 4 — Compute and Confirm the Next Spec Number

First, determine the search pattern and local directory:

- If `$SPEC_MODE` is `targeted`:
    - `$SPEC_DIR` = (`$TARGET` is `core`) ? `specs/` : `$SITES_FOLDER/$TARGET/specs/`
    - Pattern: `[0-9][0-9][0-9]-*`
- If `$SPEC_MODE` is `single`:
    - `$SPEC_DIR` = `specs/`
    - Pattern: `[0-9][0-9][0-9]-$TARGET-*`

### 4.1 — Detect Existing Prefixes

Try to collect all existing 3-digit prefixes from two sources:

1. **Remote Branches:** Run `git branch -r`. Look for branch names that end with a 3-digit number and a hyphen (e.g., `origin/001-feature`).
2. **Local Folders:** List the contents of `$SPEC_DIR` (if it exists). Look for items starting with a 3-digit number and a hyphen.

Collect all unique numbers found from these sources.

### 4.2 — Calculate and Confirm Increment

- `$SUGGESTED_NUM` = (highest number found + 1), formatted as a 3-digit zero-padded string (e.g., `1` → `001`, `12` → `012`).
- If no numbers were found, `$SUGGESTED_NUM` = `001`.

Use `vscode_askQuestions` to confirm the number or allow manual entry:

```javascript
vscode_askQuestions({
  questions: [{
    header: "next_num_choice",
    question: "Confirm the spec number prefix:",
    options: [
      { label: "$SUGGESTED_NUM", description: "Auto-incremented (recommended)", recommended: true },
      { label: "Enter manually", description: "Type a custom 3-digit prefix" }
    ],
    allowFreeformInput: false
  }]
})
```

- If the user selects "Enter manually", call `vscode_askQuestions` again to collect the number:
```javascript
vscode_askQuestions({
  questions: [{
    header: "manual_num",
    question: "Enter the 3-digit prefix for this spec (e.g. 005):",
  }]
})
```

Store the final confirmed value as `$NEXT_NUM`.

---

## STEP 5 — Ask for the Feature Name

Use `vscode_askQuestions` to ask for the feature name with a free-text input:

```
vscode_askQuestions({
  questions: [{
    header: "feature_name",
    question: "What is the name of this feature?",
    message: "Use lowercase words separated by hyphens, e.g. `user-authentication`, `product-catalog`"
  }]
})
```

Store the answer as `$FEATURE_NAME`. Normalise it: lowercase, replace spaces with hyphens, remove any characters that are not alphanumeric or hyphens.

---

## STEP 6 — Determine the Final File Path and Branch Name

Compute:

- If `$SPEC_MODE` is `targeted`:
  - `$FEATURE_DIRNAME` = `$NEXT_NUM-$FEATURE_NAME`
  - `$FEATURE_DIR` = `$SPEC_DIR/$FEATURE_DIRNAME`
  - `$SPEC_PATH` = `$FEATURE_DIR/spec.md`
  - `$BRANCH_NAME` = `$NEXT_NUM-$FEATURE_NAME`

- If `$SPEC_MODE` is `single`:
  - `$FEATURE_DIRNAME` = `$NEXT_NUM-$TARGET-$FEATURE_NAME`
  - `$FEATURE_DIR` = `$SPEC_DIR/$FEATURE_DIRNAME`
  - `$SPEC_PATH` = `$FEATURE_DIR/spec.md`
  - `$BRANCH_NAME` = `$NEXT_NUM-$TARGET-$FEATURE_NAME`

Use `vscode_askQuestions` to show a confirmation prompt before creating anything:

```
vscode_askQuestions({
  questions: [{
    header: "confirm",
    question: "Ready to create the spec — confirm the details below:",
    message: "**Mode:** `$SPEC_MODE` | **Target:** `$TARGET` | **Folder:** `$FEATURE_DIR` | **Branch:** `$BRANCH_NAME`",
    options: [
      { label: "Create spec", recommended: true },
      { label: "Cancel" }
    ],
    allowFreeformInput: false
  }]
})
```

If the user selects "Cancel", stop here with a friendly message.

---

If the user answers `no`, stop here with a friendly message.

---

## STEP 7 — Create the Spec Directory and File

1. If `$FEATURE_DIR` does not exist, create it (including any intermediate directories).
2. Create the file `$SPEC_PATH` with the template below, substituting all `{{ }}` placeholders.

---

### Spec File Template

```markdown
# {{ $FEATURE_NAME | title-case }}

**Target:** {{ $TARGET }}  
**Spec mode:** {{ $SPEC_MODE }}  
**Spec ID:** {{ $FEATURE_DIRNAME }}  
**Date:** {{ TODAY_DATE }}

---

## Overview

<!-- Describe the feature in 2–4 sentences. What does it do? Why is it needed? -->

## Goals

<!-- List the main goals this feature should achieve. Use bullet points. -->

- 

## Non-Goals

<!-- Explicitly state what this feature does NOT cover. -->

- 

## User Stories

<!-- Use the format: As a [role], I want [action] so that [benefit]. -->

- As a , I want  so that .

## Acceptance Criteria

<!-- Concrete, testable criteria. Each item should start with "Given / When / Then" or be a clear checkbox statement. -->

- [ ] 
- [ ] 

## Technical Notes

<!-- Any known constraints, dependencies, or architectural decisions relevant to this feature. Leave blank if not yet known. -->

## Open Questions

<!-- Questions that need to be resolved before or during implementation. -->

- [ ] 

## References

<!-- Links to related specs, documentation, design files, or tickets. -->

- 
```

---

## STEP 8 — Confirm and Report

After creating the file, print:

---

✅ **Spec created successfully!**

- **File:** `$SPEC_PATH`
- **Target:** `$TARGET`
- **Mode:** `$SPEC_MODE`

**Next steps:**
1. Open `$SPEC_PATH` and fill in the spec details.
2. Run `/speckit.spec-kit-multi-sites.clarify` to review the spec and resolve any ambiguities or gaps.
3. Run `/speckit.spec-kit-multi-sites.checklist` to verify the spec is implementation-ready.
4. Run `/speckit.spec-kit-multi-sites.research` to investigate technical unknowns (optional).
5. Run `/speckit.spec-kit-multi-sites.plan` to generate the technical implementation plan.
6. Run `/speckit.spec-kit-multi-sites.tasks` to generate a granular task list for development.

*Tip: To switch to a new Git branch for this feature, run:*
```
git checkout -b $BRANCH_NAME
```

---
