---
name: "aikido-security-scanner"
displayName: "Codster AIKIDO Power"
description: "Run SAST and secrets scans on code files, triage security issues, and generate periodic security reports with quantifiable metrics for compliance tracking."
keywords: ["aikido", "security", "sast", "secrets", "vulnerability", "scan"]
author: "Codster"
---

# Aikido Security Scanner

## Overview

This power integrates the Aikido MCP server to provide on-demand security scanning directly from your IDE. It detects SAST vulnerabilities, hardcoded secrets, and other security issues in your code before they reach production.

Beyond scanning, it provides a structured workflow for generating security reports with quantifiable metrics — issue counts by severity, resolution trends over time, and delta comparisons between periods. This makes it easy to demonstrate continuous security improvement during quarterly evaluations.

## Available Steering Files

- **pr-scan** — Scan all modified files in a pull request and report findings
- **file-scan** — Scan specific files or local changes for vulnerabilities
- **issues-triage** — List, review, and manage Aikido issues for a repository
- **security-report** — Generate a security snapshot with metrics and compare against previous periods
- **executive-report** — Generate a polished markdown report for evaluations, ready to paste into Google Docs

## Onboarding

### Prerequisites

- Node.js 18+ with npx available
- An Aikido account (ask your team lead for access)

### Setup

1. **Install the Power** from the Powers panel in Kiro
2. **First use:** when you run any Aikido command for the first time, Kiro will provide a sign-in link. Open it in your browser and authorize access.
3. **Done** — the session persists. No environment variables or config files to edit.

### Authentication

The Power uses Aikido's browser-based login flow. Each team member authenticates with their own Aikido account the first time they use the Power. The session is stored locally and persists across Kiro restarts.

If you need to re-authenticate (e.g., token expired), ask Kiro: "Login to Aikido"

## Common Workflows

### Quick File Scan

Ask Kiro to scan specific files:

> "Scan src/controllers/auth.controller.ts for security issues"

The agent will read the file, send it to Aikido's SAST engine, and report any findings with severity and remediation steps.

### PR Security Review

Before merging a PR, scan all changed files:

> "Scan PR #45 for security vulnerabilities"

The agent fetches the PR file list, reads each code file, runs the scan, and presents a structured report.

### Issue Triage

Review your repo's security posture:

> "Show me all open Aikido issues for ethos_beneficios_backend"

The agent queries the Aikido feed and presents issues sorted by severity with remediation guidance.

### Security Report

Generate a metrics snapshot for compliance:

> "Generate a security report for api-ethos-medic"

The agent captures the current state, compares against any previous snapshot, and outputs a report with numbers and trends.

### Executive Report

Generate a polished document for your evaluation:

> "Generate the executive security report for api-ethos-medic"

Produces a copy-paste ready markdown with security score, progress timeline, achievements, and remaining risks. Edit the "Actions Taken" section with your specific work before submitting.

## Best Practices

- Run scans before creating PRs to catch issues early
- Generate security snapshots at least monthly to track progress
- When ignoring issues, always provide a documented justification
- Review critical and high severity findings immediately
- Keep snapshots committed in the repo for historical tracking

## Troubleshooting

### MCP Server Won't Start

**Symptoms:** Tools not available, connection errors

**Solutions:**
1. Verify Node.js is installed: `node --version` (need 18+)
2. Test npx manually: `npx -y @aikidosec/mcp`
3. Check that `AIKIDO_API_KEY` environment variable is set
4. Restart Kiro after setting environment variables

### Authentication Errors

**Symptoms:** "Unauthorized" or "Invalid token" responses

**Solutions:**
1. Verify your token is valid in Aikido dashboard
2. Regenerate the token if expired
3. Ensure `AIKIDO_API_KEY` has the correct value (no extra spaces)

### No Issues Found

**Symptoms:** Scan returns empty results for a known-vulnerable file

**Solutions:**
1. Ensure the file is a supported language (.ts, .js, .py, .go, etc.)
2. Check that the repository name matches what Aikido expects
3. Verify the file content is being sent correctly (not empty)

## MCP Config

This power requires no manual configuration. Authentication is handled via browser-based login on first use.

**Troubleshooting auth issues:**
- If scan returns "sign-in required", ask Kiro: "Login to Aikido"
- Open the link Kiro provides in your browser
- After authorizing, retry your command

---

**Package:** `@aikidosec/mcp`
**MCP Server:** aikido
