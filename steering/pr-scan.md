# PR Security Scan Workflow

Scan all modified code files in a pull request for SAST vulnerabilities and hardcoded secrets.

---

## Trigger

User asks to scan a PR, for example:
- "Scan PR #45 for security issues"
- "Run security scan on pull request 110"
- "Check PR 23 for vulnerabilities"

---

## Step 1: Identify the PR

Extract the PR details from the user's request:
- **owner**: Repository owner/organization (ask if not clear)
- **repo**: Repository name (infer from workspace or ask)
- **pullNumber**: PR number from the request

If the user provides a URL like `https://github.com/org/repo/pull/45`, parse it directly.

---

## Step 2: Get Modified Files

Fetch the list of files changed in the PR using GitHub tools:

```
get PR files → filter to code files only
```

**Include:** `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.java`, `.rb`, `.php`, `.cs`, `.swift`, `.kt`

**Exclude:**
- Lock files: `package-lock.json`, `yarn.lock`, `bun.lock`, `pnpm-lock.yaml`
- Documentation: `*.md`
- Static config: `*.yml`, `*.yaml` (CI configs)
- Build outputs: `dist/`, `build/`, `.next/`
- Dependencies: `node_modules/`
- Images and assets: `*.png`, `*.jpg`, `*.svg`, `*.ico`

---

## Step 3: Read File Contents

For each filtered file, read its content:
- **Prefer local workspace** if the file exists locally at the same path
- **Fallback to GitHub** using the PR head ref if the file isn't available locally

---

## Step 4: Execute Aikido Scan

Send files to `aikido_full_scan`:

```
aikido_full_scan(
  files_to_scan: [{ relativeFilePath: "path/to/file.ts", content: "..." }],
  repository_name: "repo-name"
)
```

**Batch limit:** Maximum 50 files per request. If the PR has more than 50 code files, split into multiple batches and aggregate results.

---

## Step 5: Report Results

Present findings in this format:

```markdown
## 🔒 Security Scan: PR #{number}

**Repository:** {owner}/{repo}
**Files scanned:** {count}
**Vulnerabilities found:** {count}
**Secrets detected:** {count}

---

### 🔴 Critical (Severity > 80)

| File | Line | Issue | Remediation |
|------|------|-------|-------------|
| path/to/file.ts | 42 | Issue title | Fix description |

### 🟠 High (Severity 60-80)

| File | Line | Issue | Remediation |
|------|------|-------|-------------|
| ... | ... | ... | ... |

### 🟡 Medium (Severity 40-60)

| File | Line | Issue | Remediation |
|------|------|-------|-------------|
| ... | ... | ... | ... |

### 🟢 Low (Severity < 40)

| File | Line | Issue | Remediation |
|------|------|-------|-------------|
| ... | ... | ... | ... |

---

### ✅ No Issues

If scan returns zero findings:
"Scan complete — no vulnerabilities or secrets detected. PR is clean."
```

---

## Step 6: Evaluate False Positives

For common false positive patterns, verify before reporting:

| Finding Type | Check Before Reporting |
|--------------|----------------------|
| NoSQL Injection | Is input validated by Zod/Joi/class-validator before the query? |
| Hardcoded Secret | Is it a placeholder, test fixture, or mock value? |
| Path Traversal | Is the path built from constants + UUID, not user input? |
| SQL Injection | Are parameterized queries or an ORM used? |

If a finding is a false positive, still report it but mark it:

```
⚠️ [LIKELY FALSE POSITIVE] NoSQL Injection in auth.service.ts:55
   Reason: Input is validated by Zod schema with strict regex before reaching the query.
```

---

## Step 7: Post to PR (Optional)

If the user asks to post results to the PR:

```
Add comment to PR with the formatted scan results
```

Only do this when explicitly requested by the user.
