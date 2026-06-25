# Security Report Workflow

Generate a security snapshot with quantifiable metrics. Compare against previous snapshots to demonstrate progress over time.

---

## Trigger

User asks for a security report, for example:
- "Generate a security report for ethos_beneficios_backend"
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
- Example: `2025-01-15-ethos_beneficios_backend.json`

Sort by date descending. The most recent file is the **previous snapshot** for comparison.

---

## Step 3: Capture Current State

Query Aikido for all open issues:

```
aikido_issues_list(repo_name: "repository-name")
```

Build the snapshot object:

```json
{
  "report_date": "YYYY-MM-DD",
  "repository": "repo-name",
  "generated_by": "aikido-security-scanner-power",
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
      "severity": 85,
      "severity_label": "critical",
      "type": "sast",
      "remediation": "How to fix"
    }
  ]
}
```

### Severity Classification

| Score | Label |
|-------|-------|
| > 80 | critical |
| 60-80 | high |
| 40-59 | medium |
| < 40 | low |

### Security Score Calculation

After counting issues by severity, calculate the security score:

```
score = 100
score -= (critical_count * 3)
score -= (high_count * 2)
score -= (medium_count * 1)
score = max(score, 0)
```

Include the score in the snapshot JSON as `"security_score": <number>`.

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
