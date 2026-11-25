# Score Tracking Report Workflow

Track the security score of a repository over time, from the saved snapshots. The result is a markdown report with the score, a timeline, resolved and new issues, and the top remaining risks.

## Audience and Use

| | |
|---|---|
| **Audience** | The engineer who runs it and their team lead. Internal use only. |
| **Purpose** | Personal tracking of the security score over time, used as an evaluation metric. |
| **Contains** | Score, timeline, issue counts by severity, resolved and new issues, top risks with file and remediation. |
| **Not for** | Clients or external parties. Use `client-report.md` for those. |

The score reflects the state of the **repository**, not the work of one person. It can drop when a new vulnerability is published for a dependency nobody touched, or when Aikido changes its criteria. The report must make that clear: always keep the "Actions Taken" section, and include this note under the score table:

> Note: the score reflects the state of the repository at each date. Changes can also come from new vulnerability disclosures or changes in Aikido's evaluation criteria.

If the user only says "executive report" or "reporte ejecutivo" without more context, ask which one they want: this score tracking report, or the client report.
---

## Trigger

User asks to track their score or produce an evaluation report, for example:
- "Generate the score tracking report"
- "Create my evaluation security report"
- "Give me the security summary for my evaluation"
- "Generate the score tracking report for my-api"

---

## Step 1: Identify Repository and Period

Ask the user (if not specified):
- **Repository name**: which repo to report on
- **Period**: what time range (e.g., "Q2 2026", "last 3 months", "January to March")

If the user doesn't specify a period, use all available snapshots.

---

## Step 2: Load Snapshots

Read all snapshot files from `security/reports/` that match:
- The repository name
- The requested period (by date)

Pattern: `security/reports/YYYY-MM-DD-{repo-name}.json`

Sort by date ascending (oldest first).

---

## Step 3: Calculate Metrics

From the loaded snapshots, compute:

### Security Score (per snapshot)

```
score = 100
score -= (critical_count * 3)
score -= (high_count * 2)
score -= (medium_count * 1)
score = max(score, 0)
```

If the snapshot already has `security_score`, use it. Otherwise calculate it.

### Period Deltas

Compare first and last snapshot in the period:

```
issues_at_start = first_snapshot.summary.total_open_issues
issues_at_end = last_snapshot.summary.total_open_issues
net_change = issues_at_end - issues_at_start

score_at_start = calculated from first_snapshot
score_at_end = calculated from last_snapshot
score_change = score_at_end - score_at_start
```

### Resolved Issues

Compare issue IDs across all snapshots:
- Issues present in earlier snapshots but absent in later ones = resolved
- Issues present in later snapshots but absent in earlier ones = new

---

## Step 4: Generate Report

Output a markdown file at:
```
security/reports/score-tracking-report-{period}-{repo-name}.md
```

Example: `security/reports/score-tracking-report-Q2-2026-my-api.md`

### Report Format

```markdown
# Security Report — {Period}

**Repository:** {repo-name}
**Author:** {user or "Engineering Team"}
**Generated:** {today's date}

---

## Security Score

| Period Start | Period End | Change |
|--------------|-----------|--------|
| {score_start}/100 | {score_end}/100 | {+/-change} {↑ if positive, ↓ if negative, → if zero} |

> Note: the score reflects the state of the repository at each date. Changes can also come from new vulnerability disclosures or changes in Aikido's evaluation criteria.

---

## Summary

| Metric | Start | End | Delta |
|--------|-------|-----|-------|
| Total open issues | {n} | {n} | {delta} |
| Critical | {n} | {n} | {delta} |
| High | {n} | {n} | {delta} |
| Medium | {n} | {n} | {delta} |
| Low | {n} | {n} | {delta} |

---

## Progress Timeline

| Date | Score | Total | Critical | High | Medium | Low |
|------|-------|-------|----------|------|--------|-----|
| {date1} | {score} | {n} | {n} | {n} | {n} | {n} |
| {date2} | {score} | {n} | {n} | {n} | {n} | {n} |
| ... | ... | ... | ... | ... | ... | ... |

---

## Key Achievements

- Resolved {total_resolved} security issues during this period
- Reduced critical vulnerabilities from {start_critical} to {end_critical}
- Security score improved by {score_change} points
- {any_zero_categories — e.g., "Eliminated all leaked secrets"}

## Remaining Risks

| Priority | Issue | File | Remediation |
|----------|-------|------|-------------|
| 🔴 Critical | {title} | {file} | {remediation} |
| 🔴 Critical | {title} | {file} | {remediation} |
| ... (top 5 only) | | | |

---

## Actions Taken

- Implemented input validation on {n} endpoints
- Ignored {n} false positives with documented justification
- Scanned {n} PRs before merge
- Generated {n} security snapshots for tracking

_(Edit this section manually with your specific actions)_

---

## Next Steps

- Address remaining {critical_count} critical issues
- Focus on {most_common_type} vulnerabilities ({count} open)
- Target security score of {target} by next evaluation

_(Edit this section with your planned actions)_
```

---

## Step 5: Present to User

After saving the file, show the user:

1. The full report content in chat (so they can copy it immediately)
2. The file path where it was saved
3. A note: "The 'Actions Taken' and 'Next Steps' sections have placeholder text — edit them with your specific work before submitting."

---

## Single Snapshot (Baseline Case)

If only one snapshot exists (no comparison possible):

```markdown
# Security Report — Baseline

**Repository:** {repo-name}
**Generated:** {today's date}

---

## Security Score: {score}/100

> Note: the score reflects the state of the repository at each date. Changes can also come from new vulnerability disclosures or changes in Aikido's evaluation criteria.

---

## Current State

| Severity | Count |
|----------|-------|
| Critical | {n} |
| High | {n} |
| Medium | {n} |
| Low | {n} |
| **Total** | **{n}** |

## Top Risks

| Priority | Issue | File | Remediation |
|----------|-------|------|-------------|
| 🔴 | {title} | {file} | {fix} |
| ... (top 5) | | | |

---

## Plan

- This is the baseline measurement
- Next report will show progress against these numbers
- Priority: address {critical_count} critical issues first

_(This is your starting point. Generate another snapshot after fixing issues to demonstrate progress.)_
```

---

## Notes

- The score tracking report is meant to be **edited** by the developer before submitting — especially "Actions Taken" and "Next Steps"
- The score and metrics are auto-calculated, the narrative sections are templates
- Encourage the user to run snapshots regularly so the timeline has enough data points
- If the user asks for multi-repo, generate one section per repo in the same document
