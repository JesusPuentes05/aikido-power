# Issues Triage Workflow

List, review, and manage security issues detected by Aikido for a repository.

---

## Trigger

User asks about existing issues, for example:
- "Show me Aikido issues for ethos_beneficios_backend"
- "List security vulnerabilities in my repo"
- "What open security issues do we have?"
- "Ignore issue 12345 — it's a false positive"

---

## Flow: List Issues

### Step 1: Identify Repository

Determine the repository name:
- If specified by user, use it directly
- If not, infer from the current workspace folder name
- If ambiguous, ask the user

### Step 2: Query Aikido Feed

```
aikido_issues_list(repo_name: "repository-name")
```

Optional filters the user might specify:
- `issue_types`: filter by type (e.g., "sast", "leaked_secret", "open_source")
- `repo_branch_name`: filter by branch

### Step 3: Present Results

Format as a sorted table, critical issues first:

```markdown
## 🔒 Open Security Issues: {repo_name}

**Total:** {count} issues

| # | ID | Severity | Type | Title | Remediation |
|---|-----|----------|------|-------|-------------|
| 1 | {id} | 🔴 Critical ({score}) | SAST | {title} | {remediation} |
| 2 | {id} | 🟠 High ({score}) | Secret | {title} | {remediation} |
| 3 | {id} | 🟡 Medium ({score}) | Dependency | {title} | {remediation} |
| ... | ... | ... | ... | ... | ... |

### Summary by Severity
- 🔴 Critical: {count}
- 🟠 High: {count}
- 🟡 Medium: {count}
- 🟢 Low: {count}

### Summary by Type
- SAST: {count}
- Open Source: {count}
- Leaked Secret: {count}
- IaC: {count}
```

---

## Flow: Ignore Issue

### Step 1: Get Issue ID and Reason

The user must provide:
- **Issue ID** — the Aikido issue identifier
- **Reason** — justification for ignoring (required)

If missing, ask:
- "What's the issue ID you want to ignore?"
- "What's the reason for ignoring it? (e.g., false positive, accepted risk, test code)"

### Step 2: Execute Ignore

```
aikido_ignore_issue(issue_id: "12345", reason: "False positive — input validated by Zod schema before query")
```

### Step 3: Confirm

```markdown
✅ Issue #{id} ignored.
**Reason:** {reason}

The issue will no longer appear in future scans or reports.
```

---

## Flow: Review Specific Issue

If user asks about a specific issue for context:

1. Locate the issue in the list by ID or title
2. Present full details: severity, type, file location, remediation
3. If the file is in the workspace, read the relevant code section
4. Assess whether it's a true positive or false positive based on surrounding code
5. Recommend action: fix, ignore with reason, or escalate

---

## Guidelines for Triage Recommendations

When helping users prioritize:

| Severity | Recommended Action |
|----------|-------------------|
| Critical (>80) | Fix immediately or escalate to team lead |
| High (60-80) | Fix within current sprint |
| Medium (40-60) | Add to backlog, fix when touching that code |
| Low (<40) | Track, fix opportunistically |

For false positives, always recommend ignoring with a documented reason rather than leaving them open.
