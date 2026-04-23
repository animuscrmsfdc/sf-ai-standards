Deploy the current changes to the Salesforce org: $ARGUMENTS

Default runs the full validation pipeline (recommended for pre-PR and release deploys).
Use `--quick` to skip the weakness scan and deploy immediately (iterative development only).

If a specific component is provided in $ARGUMENTS (alongside or instead of a flag), deploy only that component.
If no argument is provided, deploy the entire force-app source directory.

---

## Step 1 — Collect changed files

Run all three commands and combine into a single deduplicated list of file paths:

```
git diff --name-only
git diff --cached --name-only
git ls-files --others --exclude-standard
```

If no files are found, report "Nothing to deploy" and stop.

---

## Step 2 — LWC lint check

If any changed files are under `lwc/`, run:

```
npm run lint
```

If lint errors are found, stop and report them — do not proceed.

---

## Step 2b — Quick mode check

If `$ARGUMENTS` contains `--quick`, skip Steps 3, 4, and 5 entirely and proceed directly to Step 6.

---

## Step 3 — Weakness scan (scoped to changed files)

Run the find-weaknesses agent, but **scope the analysis exclusively to the changed files identified in Step 1**. Only report findings for those files.

**Pre-process findings before scoring:**
- **Discard** any MEDIUM or LOW/INFO/other finding whose file path ends in `.md`.
- **Reclassify severity**: A finding is CRITICAL only if it has a real, demonstrable impact in production (security vulnerability, data loss risk, broken logic, missing sharing enforcement, SOQL injection, unhandled exception in a critical path). Style suggestions, documentation notes, or purely informational findings must be downgraded to HIGH or MEDIUM — never left as CRITICAL.

**Calculate Tech Debt Score (0–10):**
- Weights: CRITICAL = 10, HIGH = 5, MEDIUM = 3, LOW/INFO/other = 1
- Formula: `score = sum(weight_i for every finding i) / totalFindings`
- If no findings, score = 0. Round to one decimal place.
- Example: 1 CRITICAL + 2 HIGH + 4 MEDIUM + 3 LOW = 10 findings → (10+5+5+3+3+3+3+1+1+1)/10 = 3.5

---

## Step 4 — Display findings summary

Assign sequential 4-digit numbers per severity group (CRITICAL first, then HIGH, then MEDIUM), resetting to 0001 per group. Omit buckets with no findings.

```
[CRITICAL-0001] <file>:<line> — <description>
[HIGH-0001] <file>:<line> — <description>
[MEDIUM-0001] <file>:<line> — <description>
```

If no findings at all, print `No issues found in changed files.`

Then display the score card:

```
┌──────────────────────────────────────────────┐
│  Tech Debt Score: <score>/10                 │
│  Critical: <n>  High: <n>  Medium: <n>  Other: <n>  Total: <n> │
└──────────────────────────────────────────────┘
```

---

## Step 5 — Gate: cancel on CRITICAL or high score

**If any CRITICAL findings exist**, cancel and display:

```
Deploy cancelled. 1 or more CRITICAL findings must be resolved before deploying.
```

Stop here — do not proceed.

**If the Tech Debt Score is greater than 7**, cancel and display:

```
Deploy cancelled. Tech Debt Score (<score>/10) exceeds the threshold of 7.
Resolve the high/critical weaknesses reported above before deploying.
```

Stop here — do not proceed.

---

## Step 6 — Ask the user to continue

Ask: "Tech Debt Score is <score>/10. Do you want to continue with the commit and deploy? (yes/no)"

Wait for the user's response. If the user answers anything other than "yes", cancel and stop.

---

## Step 7 — Show git status

Run `git status` to show the user what files will be staged.

---

## Step 8 — Ask for commit message

Ask: "Enter your commit message:"

Wait for the user's input before proceeding.

---

## Step 9 — Stage files

Before staging, filter out any path that matches a sensitive-file pattern (`.env`, `*.key`, `*.pem`, `*.p12`, `*.pfx`, any filename containing `secret`, `credential`, or `token`). If any file is filtered out, display a warning listing the skipped paths.

Stage the remaining files individually — do not use `git add -A`:

```
git add <file1> <file2> ...
```

---

## Step 10 — Commit

```
git commit -m "<user-provided message>

Co-Authored-By: Claude Code <noreply@anthropic.com>"
```

---

## Step 11 — Deploy

Show the user what will be deployed by running `git diff --stat HEAD~1`.

**If $ARGUMENTS is empty**, run:

```
sf project deploy start --source-dir force-app --target-org $SF_TARGET_ORG
```

**If $ARGUMENTS contains a metadata type and name** (e.g. `CustomObject:MyObj__c`), run:

```
sf project deploy start --metadata <value> --target-org $SF_TARGET_ORG
```

**If $ARGUMENTS is "test"**, run the full deploy then also run:

```
sf apex run test --target-org $SF_TARGET_ORG --code-coverage --result-format human
```

Report test results and coverage. Fail if any test fails.

---

## Step 12 — Confirm deploy status

```
sf project deploy report
```

If there are errors:
- Read each error message carefully.
- Identify the file and line number.
- Suggest the fix and apply it if straightforward.
- Re-run the deploy after fixing.

---

## Step 13 — On successful deploy, update memory

a. Get current branch:

```
git branch --show-current
```

b. **If the branch starts with `hotfix/`:**
   - Extract the slug (everything after `hotfix/`, e.g. `hotfix/fix-null-pointer` → `fix-null-pointer`).
   - Determine the project memory path: `echo ~/.claude/projects/$(pwd | sed 's|/|-|g' | sed 's|^-||')/memory/project_context.md`
   - Under `### Hotfixes` in that file, append:
     `- **<slug>** — deployed to $SF_TARGET_ORG on <today's date ISO 8601>.`
   - Check `### Shipped` for any entry whose spec path contains the slug as a substring. If found, append to that entry:
     `Hotfix applied: <slug> (<today's date ISO 8601>).`
   - If no Shipped entry matches, add under `### Hotfixes` only — do not prompt the user.

c. **If the branch starts with `feature/`:**
   - Extract the slug (everything after `feature/`, e.g. `feature/order-management` → `order-management`).
   - Determine the project memory path: `echo ~/.claude/projects/$(pwd | sed 's|/|-|g' | sed 's|^-||')/memory/project_context.md`
   - Find the entry under `### In Progress` whose spec path is `specs/<slug>.md`.
   - If found: remove it from `### In Progress`, append `Shipped: <today's date ISO 8601>.` to the entry, and add it under `### Shipped`.
   - If not found (incremental deploy of an already-shipped feature), do nothing.

d. **For any other branch pattern**, do nothing to the memory file.

Do not update the memory file if the deploy failed.
