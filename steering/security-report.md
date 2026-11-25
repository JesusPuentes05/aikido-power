# Security Report Workflow

Generate a security snapshot with quantifiable metrics. Compare against previous snapshots to demonstrate progress over time.

---

## Trigger

User asks for a security report, for example:
- "Generate a security report for my-backend"
- "Create a security snapshot"
- "Show me our security progress this quarter"
- "Generate the quarterly security metrics"

---

## Step 1: Identify Repository and Report Directory

Determine:
- **Repository name**: from user request or current workspace
- **Reports directory**: `security/reports/` in the workspace root

If the `security/reports/` directory doesn't exist, create it.

---

## Step 2: Check for Previous Snapshots

Look for existing snapshot files in `security/reports/`:
- Pattern: `YYYY-MM-DD-{repo-name}.json`
- Example: `2025-01-15-my-backend.json`

Sort by date descending. The most recent file is the **previous snapshot** for comparison.

---

## Step 3: Capture Current State

Query Aikido for all open issues:

```
aikido_issues_list(repo_name: "repository-name", page: 0)
```

**Fetch every page.** Keep requesting `page: 1, 2, 3...` until a page returns no issues. A snapshot built from only the first page has wrong totals.

Each issue includes `location.branch_name`. If the issues of one repository come from more than one branch, ask the user which branch to snapshot (use the `repo_branch_name` filter) and never mix branches in one snapshot.

Build the snapshot object:

```json
{
  "report_date": "YYYY-MM-DD",
  "repository": "repo-name",
  "branch": "branch-name",
  "generated_by": "aikido-power",
  "summary": {
    "total_open_issues": 0,
    "critical": 0,
    "high": 0,
    "medium": 0,
    "low": 0
  },
  "by_type": {
    "sast": 0,
    "leaked_secret": 0,
    "open_source": 0,
    "iac": 0,
    "malware": 0,
    "eol": 0
  },
  "issues": [
    {
      "id": "issue_id",
      "title": "Issue title",
      "severity": 82,
      "severity_label": "high",
      "type": "open_source",
      "remediation": "How to fix",
      "file": "path/or/null",
      "line": null,
      "package": "package-name or null",
      "installed_version": "version or null",
      "cve": "CVE or Aikido id, or null"
    }
  ]
}
```

Field mapping from `aikido_issues_list`: `id` = `issue_id`, `title` = `issue_title`, `severity` = `issue_severity`, `severity_label` = `issue_severity_label`, `type` = `issue_type`, `remediation` = `issue_remediation`, `file` = `issue_file`, `package` = `issue_affected_package`, `installed_version` = `issue_installed_version`, `cve` = `issue_cve_id`. Use `null` when a field is not returned. Never store secret values.

### Severity Classification

**Use the label Aikido returns** (`issue_severity_label`: `critical`, `high`, `medium`, `low`). Do not derive the label from the numeric score with fixed thresholds. Aikido's boundaries do not match round numbers: for example, issues with a score of 82 are labeled `high`, not `critical`.

If an issue has no label, count it as "unclassified" and tell the user. Do not guess.

### Security Score Calculation

After counting issues by label, calculate the security score:

```
score = 100
score -= (critical_count * 3)
score -= (high_count * 2)
score -= (medium_count * 1)
score = max(score, 0)
```

Include the score in the snapshot JSON as `"security_score": <number>`.

**When reading older snapshots, always recalculate the score from the severity counts** with this formula. Do not trust a stored `security_score`: older snapshots may have been written with a different formula and would mix two scales in one timeline.

---

## Step 4: Save Snapshot

Write the JSON file to:
```
security/reports/YYYY-MM-DD-{repo-name}.json
```

Use today's date. If a file with today's date already exists, overwrite it (re-running the report on the same day replaces the previous run).

---

## Step 5: Compare with Previous Snapshot

If a previous snapshot exists, calculate deltas:

### Delta Calculation

```
resolved_issues = issues in previous NOT in current (by ID)
new_issues = issues in current NOT in previous (by ID)
persistent_issues = issues in BOTH (still open)
```

### Metrics

```
net_change = current.total - previous.total
critical_change = current.critical - previous.critical
high_change = current.high - previous.high
```

---

## Step 6: Generate Report

### If Previous Snapshot Exists (Comparison Report)

```markdown
## 🔒 Security Report: {repo-name}

**Date:** {today}
**Previous snapshot:** {previous_date}
**Period:** {days between snapshots} days

---

### Summary

| Metric | Previous | Current | Delta |
|--------|----------|---------|-------|
| Total open issues | {prev} | {curr} | {delta} {✅ if negative, ⚠️ if positive} |
| Critical | {prev} | {curr} | {delta} |
| High | {prev} | {curr} | {delta} |
| Medium | {prev} | {curr} | {delta} |
| Low | {prev} | {curr} | {delta} |

### By Type

| Type | Previous | Current | Delta |
|------|----------|---------|-------|
| SAST | {prev} | {curr} | {delta} |
| Open Source | {prev} | {curr} | {delta} |
| Leaked Secret | {prev} | {curr} | {delta} |
| IaC | {prev} | {curr} | {delta} |

### Progress

- ✅ Issues resolved: {count}
- ⚠️ New issues introduced: {count}
- 📊 Net change: {net_change}

### Resolved Since Last Snapshot

{For each resolved issue:}
- ✅ [{severity_label}] {title}

### New Issues Since Last Snapshot

{For each new issue:}
- ⚠️ [{severity_label}] {title} — {remediation}

### Top Priority (Remaining)

{Top 5 open issues by severity:}
1. 🔴 [{severity}] {title} — {remediation}
2. 🟠 [{severity}] {title} — {remediation}
3. ...

---

📁 Snapshot saved to: `security/reports/{filename}.json`
```

### If No Previous Snapshot (Baseline Report)

```markdown
## 🔒 Security Report: {repo-name} (Baseline)

**Date:** {today}
**Status:** First snapshot — no previous data to compare

---

### Current State

| Severity | Count |
|----------|-------|
| 🔴 Critical | {count} |
| 🟠 High | {count} |
| 🟡 Medium | {count} |
| 🟢 Low | {count} |
| **Total** | **{count}** |

### By Type

| Type | Count |
|------|-------|
| SAST | {count} |
| Open Source | {count} |
| Leaked Secret | {count} |
| IaC | {count} |

### All Open Issues

{For each issue, sorted by severity:}
1. 🔴 [{severity}] {title} — {remediation}
2. 🟠 [{severity}] {title} — {remediation}
3. ...

---

ℹ️ This is your baseline snapshot. Run another report in the future to track progress.

📁 Snapshot saved to: `security/reports/{filename}.json`
```

---

## Step 7: Quarter Summary (Optional)

If the user asks for a quarterly or period summary (e.g., "show me Q1 progress"):

1. Find all snapshots within the requested period
2. Build a timeline table:

```markdown
## 📊 Security Progress: {period}

### Timeline

| Date | Total | Critical | High | Medium | Low | Resolved | New |
|------|-------|----------|------|--------|-----|----------|-----|
| {date1} | {n} | {n} | {n} | {n} | {n} | — | — |
| {date2} | {n} | {n} | {n} | {n} | {n} | {n} | {n} |
| {date3} | {n} | {n} | {n} | {n} | {n} | {n} | {n} |

### Period Summary

- **Started with:** {first snapshot total} issues ({critical} critical)
- **Ended with:** {last snapshot total} issues ({critical} critical)
- **Total resolved:** {sum of resolved across all periods}
- **Total introduced:** {sum of new across all periods}
- **Net improvement:** {first total - last total} issues
```

This gives evaluators a clear picture of continuous improvement within a quarter.

---

## Notes

- Snapshots use Aikido issue IDs for tracking, so renames or re-detections won't cause false "resolved" counts
- If an issue is ignored in Aikido, it won't appear in future snapshots (correctly counted as resolved)
- The reports directory should be committed to the repo so the team has shared visibility
- Any team member can run the report — it queries the shared Aikido workspace
