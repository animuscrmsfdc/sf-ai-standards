Initialize this project's Claude Code environment by collecting org details and replacing all placeholders.

---

## Step 1 — Collect project details

Ask the user the following questions one at a time. Wait for each answer before asking the next.

1. "What is your Salesforce org alias? (e.g. `my-sandbox`)"
2. "What is your sandbox login username? (e.g. `user@domain.com.sandbox`)"
3. "What is your project name? (e.g. `MYPROJECT`)"

Store the answers as:
- `<ORG_ALIAS>` — the org alias
- `<SANDBOX_USERNAME>` — the sandbox username
- `<PROJECT_NAME>` — the project name

---

## Step 2 — Confirm before writing

Display a summary:

```
Project name:      <PROJECT_NAME>
Org alias:         <ORG_ALIAS>
Sandbox username:  <SANDBOX_USERNAME>
```

Ask: "Apply these values? (yes/no)"

Wait for confirmation. If anything other than "yes", cancel and stop.

---

## Step 3 — Update CLAUDE.md

In `CLAUDE.md`, replace:
- Every occurrence of `<YOUR_ORG_ALIAS>` → `<ORG_ALIAS>`
- Every occurrence of `<YOUR_SANDBOX_USERNAME>` → `<SANDBOX_USERNAME>`
- Every occurrence of `<YOUR_PROJECT_NAME>` → `<PROJECT_NAME>`

---

## Step 4 — Write .claude/settings.local.json

Write `.claude/settings.local.json` with the following content, replacing `<ORG_ALIAS>` with the value collected in Step 1:

```json
{
  "env": {
    "SF_TARGET_ORG": "<ORG_ALIAS>",
    "SF_LOG_LEVEL": "warn"
  },
  "permissions": {
    "allow": [
      "Bash(sf org:*)",
      "Bash(python3 -c \"import sys,json; d=json.load\\(sys.stdin\\); r=d.get\\(''''result'''',{}\\); print\\(''''Status:'''', r.get\\(''''status''''\\)\\); [print\\(''''ERROR:'''', e.get\\(''''fileName'''',''''''''\\), e.get\\(''''problem'''',''''''''\\)\\) for e in r.get\\(''''details'''',{}\\).get\\(''''componentFailures'''',[]\\)]\")",
      "WebFetch(domain:github.com)",
      "WebFetch(domain:raw.githubusercontent.com)",
      "WebSearch",
      "WebFetch(domain:developer.salesforce.com)",
      "Bash(git config:*)",
      "Bash(git credential:*)"
    ]
  }
}
```

---

## Step 5 — Confirm and show next steps

Tell the user:

```
Setup complete.

Files updated:
  CLAUDE.md              — placeholders replaced
  .claude/settings.local.json — SF_TARGET_ORG set to <ORG_ALIAS>

Next steps:
  1. Verify your org is authenticated: sf org display --target-org <ORG_ALIAS>
  2. If not yet authenticated: sf org login web --alias <ORG_ALIAS>
  3. Run /deploy or /run-tests to confirm the connection works.
```
