# Aikido Security Scanner (Kiro Power)

A [Kiro Power](https://kiro.dev) that connects Kiro to [Aikido Security](https://www.aikido.dev). It lets you scan code for SAST vulnerabilities and hardcoded secrets, triage Aikido issues, and generate security reports with metrics you can use in compliance reviews or performance evaluations.

## What it does

| Workflow | Steering file | Description |
|----------|---------------|-------------|
| File scan | `steering/file-scan.md` | Scan specific files, a directory, or your local git changes (staged and unstaged). |
| PR scan | `steering/pr-scan.md` | Scan every code file modified in a pull request. Optionally posts the results as a PR comment. |
| Issue triage | `steering/issues-triage.md` | List open Aikido issues for a repository, review a single issue, or ignore one with a documented reason. |
| Security report | `steering/security-report.md` | Save a JSON snapshot of the current issues and compare it with the previous one (resolved, new, persistent issues and a security score). |
| Executive report | `steering/executive-report.md` | Build a markdown report from the saved snapshots, ready to paste into Google Docs or Confluence. |

The scan workflows also flag likely false positives, such as input already validated by Zod/Joi, placeholder secrets, or paths built from constants.

## Repository layout

```
aikido-power/
├── POWER.md      Power definition, onboarding and troubleshooting
├── mcp.json      MCP server configuration (@aikidosec/mcp)
├── README.md     This file
└── steering/     One workflow guide per capability
```

## Requirements

- **Kiro** with Powers support.
- **Node.js 18 or newer**, with `npx` available. The MCP server runs as `npx -y @aikidosec/mcp`.
- **An Aikido account** with access to the repositories you want to scan.
- **Network access** to Aikido and to the npm registry.
- **Optional, for PR scans:** a GitHub MCP server or equivalent tools configured in Kiro. This Power does not install one. Without it, PR scans cannot fetch the list of changed files.
- **Optional, for reports:** a git repository where `security/reports/` can be created and committed.

## Installation

1. Open the Powers panel in Kiro and install this Power from this repository (a local path or Git URL).
2. Confirm Node.js is available:
   ```
   node --version
   ```
3. Ask Kiro to run any Aikido command, for example "Scan src/app.ts for security issues".

## Authentication

On first use, Kiro shows a sign-in link. Open it in your browser and authorize access. The session is stored locally. To sign in again, ask Kiro: "Login to Aikido".

If the MCP server reports an unauthorized or invalid token error, check whether your Aikido setup requires an API key. In that case set the `AIKIDO_API_KEY` environment variable and restart Kiro. Do not commit the key to the repository.

## Usage examples

```
Scan src/controllers/auth.controller.ts for security issues
Check my staged changes for vulnerabilities
Scan PR #45 for security vulnerabilities
Show me all open Aikido issues for my-backend
Ignore issue 12345, it is a false positive validated by a schema
Generate a security report for my-api
Generate the executive security report for my-api
```

## Security report data

Snapshots are written to `security/reports/` in the workspace root:

- `YYYY-MM-DD-{repo-name}.json`: one snapshot per day. Running the report again on the same day overwrites it.
- `executive-report-{period}-{repo-name}.md`: the generated executive report.

The security score is `100 - (3 × critical) - (2 × high) - (1 × medium)`, with a minimum of 0. Low severity issues do not affect it.

Commit the `security/reports/` folder so the team shares the history. Snapshots contain issue titles and file paths, so check that your repository's visibility is appropriate for that.

## Troubleshooting

| Problem | What to try |
|---------|-------------|
| Aikido tools do not appear | Run `node --version` (need 18+), then `npx -y @aikidosec/mcp` manually to see the error. Restart Kiro. |
| "Unauthorized" or "sign-in required" | Ask Kiro "Login to Aikido". If that fails, check `AIKIDO_API_KEY` for extra spaces or an expired token. |
| Scan returns nothing for a vulnerable file | Check that the language is supported, that the repository name matches the one in Aikido, and that the file is not empty. |
| PR scan cannot list files | Configure a GitHub MCP server in Kiro. |

## Notes

- Ignoring an issue in Aikido removes it from future scans and snapshots, and the report counts it as resolved. Always give a documented reason.
- The executive report includes "Actions Taken" and "Next Steps" sections with placeholder text. Edit them before submitting.
