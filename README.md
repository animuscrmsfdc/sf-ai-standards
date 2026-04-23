```
==============================================================================
  ____    _    _     _____ ____  _____ ___  ____   ____  _____
 / ___|  / \  | |   | ____/ ___||  ___/ _ \|  _ \ / ___|| ____|
 \___ \ / _ \ | |   |  _| \___ \| |_ | | | | |_) | |   |  _|
  ___) / ___ \| |___| |___ ___) |  _|| |_| |  _ <| |___| |___
 |____/_/   \_\_____|_____|____/|_|   \___/|_| \_\\____|_____|

      _      ___      ____  _____  _    _   _ ____    _    ____  ____   ____
     / \    |_ _|    / ___||_   _|/ \  | \ | |  _ \  / \  |  _ \|  _ \ / ___|
    / _ \    | |     \___ \  | | / _ \ |  \| | | | |/ _ \ | |_) | | | |\___ \
   / ___ \   | |      ___) | | |/ ___ \| |\  | |_| / ___ \|  _ <| |_| | ___) |
  /_/   \_\ |___|    |____/  |_/_/   \_\_| \_|____/_/   \_\_| \_\____/ |____/
==============================================================================
```

# sf-ai-standards

Team template for standardizing Claude Code and VS Code environments across Salesforce development projects.

## Purpose

Copy this repo's `.claude/` folder into any Salesforce project to get a consistent, secure, and pre-configured AI coding agent setup — including slash commands, agents, skills, and permission gates.

## What's included

| Path                          | Purpose                                                                       |
| ----------------------------- | ----------------------------------------------------------------------------- |
| `CLAUDE.md`                   | Root AI context: coding standards, security rules, testing requirements       |
| `CLAUDE.local.md`             | Template for personal local overrides (gitignored)                            |
| `.claude/settings.json`       | Shared Claude Code permissions and allowed CLI commands                       |
| `.claude/settings.local.json` | Personal permission overrides and env vars (gitignored)                       |
| `.claude/commands/`           | Slash commands for multi-step orchestrated workflows                          |
| `.claude/agents/`             | Subagents: business analyst (requirements), find-weaknesses (security review) |
| `.claude/skills/`             | Domain knowledge for scaffolding any Salesforce metadata or component         |
| `.vscode/`                    | Recommended VS Code extensions and workspace settings                         |
| `config/ai-rules/`            | Machine-readable Apex coding rules for AI enforcement                         |
| `.github/CODEOWNERS`          | Review requirements by file path                                              |
| `.github/SECURITY.md`         | Vulnerability disclosure policy                                               |

## Quick start

1. Clone this repo:

   ```
   git clone <this-repo-url>
   ```

2. Copy `.claude/` into your Salesforce project root:

   ```
   cp -r sf-ai-standards/.claude/ your-project/
   ```

3. Copy `CLAUDE.md` and `CLAUDE.local.md` into your project root:

   ```
   cp sf-ai-standards/CLAUDE.md your-project/
   cp sf-ai-standards/CLAUDE.local.md your-project/
   ```

4. Open Claude Code in your project root and run:

   ```
   /setup
   ```

   This will prompt for your org alias, sandbox username, and project name, then update `CLAUDE.md` and `.claude/settings.local.json` automatically.

5. Open VS Code in your project — accept the recommended extensions prompt.

## Slash commands

Commands handle multi-step workflows with decision gates, side effects, or confirmation prompts. For metadata scaffolding (objects, fields, classes, components), just ask Claude directly — the skills handle it.

| Command        | Description                                                                      |
| -------------- | -------------------------------------------------------------------------------- |
| `/setup`       | First-time setup: replace placeholders in CLAUDE.md and configure org alias      |
| `/new-feature` | Structured requirements interview → spec file → feature branch                   |
| `/deploy`      | Weakness scan + commit + deploy with tech debt gate (use `--quick` to skip scan) |
| `/run-tests`   | Run Apex tests with coverage report and failure analysis                         |
| `/retrieve`    | Retrieve metadata from org                                                       |
| `/changelog`   | Generate CHANGELOG.md entry from git diff                                        |

### When to add a new command

Only create a command when the workflow has **multiple steps, requires user confirmation at a gate, or produces side effects** beyond file scaffolding (branch creation, memory updates, deploy pipeline). If the task is "create X", it belongs in a skill.

## Skills

Skills are the scalable layer. Claude reads them as context and applies them to any natural language request — no command file needed per metadata type.

| Skill                       | Covers                                                                                                                          |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `salesforce-admin`          | Objects, fields (12+ types), validation rules, platform events, page layouts, list views, global value sets, permission set FLS |
| `salesforce-developer-apex` | Triggers (static switch pattern), Apex class templates, platform event publishing, security enforcement                         |
| `salesforce-lwc`            | Component scaffolding, wire adapters, parent-child communication, Jest testing, security, accessibility                         |
| `salesforce-test-overwatch` | Test execution, failure analysis, fix planning with approval gate, test class authoring standards                               |
| `salesforce-code-analyzer`  | PMD static analysis, violation triage, ruleset management                                                                       |
| `salesforce-devops`         | package.xml sync rules, deployment strategies, CI/CD pipeline reference                                                         |

### When to add a new skill

Add or extend a skill when a new Salesforce feature area needs coverage (Flows, Named Credentials, Experience Cloud, etc.). Skills scale — a new skill covers all metadata types in that area without adding command files.

## Standards enforced

- `with sharing` on all Apex classes by default
- `Security.stripInaccessible()` on all @AuraEnabled write methods
- No hardcoded IDs — Custom Metadata or Custom Labels only
- No `Database.query()` with string concatenation — bind variables required
- 90% minimum code coverage
- One trigger per object, static switch-based handler pattern
- SHA-256 minimum for any cryptographic operations
- No `eval()` or `innerHTML` in LWC JavaScript
