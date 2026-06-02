---
mode: agent
description: Create a spec in the right site or core folder with per-site auto-incremented numbering
---

# speckit.spec-kit-multi-sites.specify

You are running the **Multi-Sites Specify** command. This is a multi-site-aware replacement for `/speckit.specify`. Follow every step below exactly and in order. Do not skip steps. Do not proceed to a later step until the user has responded to the current one.

---

## STEP 1 — Detect the Sites Folder

Scan the project's directory structure to find the folder that contains the individual website sub-folders.

Apply the following detection rules, in priority order:

1. **Drupal multi-site** — look for `docroot/sites/` or `web/sites/` or `sites/`. A Drupal sites folder contains sub-directories that each have a `settings.php` or `settings.local.php` file inside them.
2. **Generic CMS / framework** — look for any folder named `sites/`, `webs/`, `domains/`, `tenants/`, `apps/`, or `vhosts/` at the root or one level deep that contains at least two sub-directories.
3. **Custom** — if no candidate is found automatically, set `detected_sites_folder` to `null`.

After running the scan, present the result to the user like this:

---

**Sites folder detection**

> I analysed the project structure and found: **`<detected_sites_folder>`** *(or "no sites folder detected" if null)*

Please confirm or provide the correct path:

- If the suggestion is correct, just press **Enter** or type `yes`.
- Otherwise, type the correct path relative to the project root (e.g. `docroot/sites`).

---

Store the confirmed value as `$SITES_FOLDER`.

---

## STEP 2 — Choose the Spec Mode

Ask the user to choose how specs should be organised:

---

**Spec mode selection**

How would you like to organise your specs?

**1 — Targeted specs** *(recommended for large multi-site projects)*
Each website has its own `specs/` folder inside it. Auto-increment numbers are independent per website (`websiteA/specs/001-…`, `websiteA/specs/002-…` vs `websiteB/specs/001-…`). Core specs live in a root-level `specs/` folder with their own independent counter.

**2 — Single specs**
All specs share one `specs/` folder at the root level. The website or core name is embedded in the file name so each scope has its own independent counter (`001-core-…`, `001-websiteA-…`, `001-websiteB-…`).

Type `1` or `2` (or the full label).

---

Store the choice as `$SPEC_MODE` (value: `targeted` or `single`).

---

## STEP 3 — Choose the Target (Website or Core)

List the immediate sub-directories inside `$SITES_FOLDER`. Exclude any directory named `all`, `default`, `.git`, `node_modules`, or any file (not folder). Sort alphabetically.

Also add a special option called **`core`** at the top of the list.

Present the options like this:

---

**Target selection**

Where should this spec be created?

- `core` — shared core feature (top-level, applies to all websites)
- `<website-1>`
- `<website-2>`
- *(… all sub-directories found …)*

Type the name of the target.

---

Store the answer as `$TARGET`.

---

## STEP 4 — Compute the Next Spec Number

Use the following logic to determine the next sequential number `$NEXT_NUM` for the chosen `$TARGET`:

### If $SPEC_MODE is `targeted`:

- If `$TARGET` is `core`:
  - Look in `specs/` at the project root.
  - Count all files whose name matches the pattern `NNN-*` (where `NNN` is a zero-padded 3-digit number).
  - `$NEXT_NUM` = (count of matching files) + 1
  - `$SPEC_DIR` = `specs/`

- If `$TARGET` is a website name:
  - Look in `$SITES_FOLDER/$TARGET/specs/`.
  - Count all files whose name matches the pattern `NNN-*`.
  - `$NEXT_NUM` = (count of matching files) + 1
  - `$SPEC_DIR` = `$SITES_FOLDER/$TARGET/specs/`

### If $SPEC_MODE is `single`:

- `$SPEC_DIR` = `specs/` (always at project root)
- Scan all files in `specs/` whose name matches `NNN-<$TARGET>-*` (case-insensitive).
- `$NEXT_NUM` = (count of those matching files) + 1

In both modes, format `$NEXT_NUM` as a zero-padded 3-digit string (e.g. `1` → `001`, `12` → `012`).

---

## STEP 5 — Ask for the Feature Name

Ask the user:

---

**Feature name**

What is the name of this feature? *(Use lowercase words separated by hyphens, e.g. `user-authentication`, `product-catalog`)*

---

Store the answer as `$FEATURE_NAME`. Normalise it: lowercase, replace spaces with hyphens, remove any characters that are not alphanumeric or hyphens.

---

## STEP 6 — Determine the Final File Path and Branch Name

Compute:

- If `$SPEC_MODE` is `targeted`:
  - `$SPEC_FILENAME` = `$NEXT_NUM-$FEATURE_NAME.md`
  - `$SPEC_PATH` = `$SPEC_DIR/$SPEC_FILENAME`
  - `$BRANCH_NAME` = `$NEXT_NUM-$FEATURE_NAME` *(for targeted mode, no scope prefix in branch)*

- If `$SPEC_MODE` is `single`:
  - `$SPEC_FILENAME` = `$NEXT_NUM-$TARGET-$FEATURE_NAME.md`
  - `$SPEC_PATH` = `$SPEC_DIR/$SPEC_FILENAME`
  - `$BRANCH_NAME` = `$NEXT_NUM-$TARGET-$FEATURE_NAME`

Show a summary to the user before creating anything:

---

**Summary — about to create**

| Field          | Value                      |
|----------------|----------------------------|
| Mode           | `$SPEC_MODE`               |
| Target         | `$TARGET`                  |
| Specs folder   | `$SPEC_DIR`                |
| Spec file      | `$SPEC_FILENAME`           |
| Full path      | `$SPEC_PATH`               |
| Git branch     | `$BRANCH_NAME`             |

Proceed? Type `yes` to create the spec, or `no` to cancel.

---

If the user answers `no`, stop here with a friendly message.

---

## STEP 7 — Create the Spec Directory and File

1. If `$SPEC_DIR` does not exist, create it (including any intermediate directories).
2. Create the file `$SPEC_PATH` with the template below, substituting all `{{ }}` placeholders.

---

### Spec File Template

```markdown
# {{ $FEATURE_NAME | title-case }}

**Target:** {{ $TARGET }}  
**Spec mode:** {{ $SPEC_MODE }}  
**Spec ID:** {{ $SPEC_FILENAME }}  
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
2. When ready, run `/speckit.clarify` to resolve ambiguities.
3. Then run `/speckit.plan` to generate the technical plan.

*Tip: To switch to a new Git branch for this feature, run:*
```
git checkout -b $BRANCH_NAME
```

---
