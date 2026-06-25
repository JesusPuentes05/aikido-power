# File Security Scan Workflow

Scan specific files or local changes for SAST vulnerabilities and hardcoded secrets.

---

## Trigger

User asks to scan files, for example:
- "Scan src/controllers/auth.controller.ts for security issues"
- "Run security scan on the services folder"
- "Check my staged changes for vulnerabilities"
- "Scan the files I just modified"

---

## Mode Detection

Determine the scan mode from the user's request:

| Request Pattern | Mode |
|----------------|------|
| Specific file(s) mentioned | **File Scan** |
| Directory mentioned | **Directory Scan** |
| "staged changes" / "my changes" / "local changes" | **Git Changes Scan** |

---

## Mode: File Scan

### Step 1: Identify Files

Collect the file paths from the user's request. Accept:
- Single file: `src/controllers/auth.controller.ts`
- Multiple files: `src/services/payment.service.ts, src/services/auth.service.ts`
- Glob-like: `src/controllers/*.ts` (expand to actual files)

### Step 2: Read and Scan

Read each file's content and send to Aikido:

```
aikido_full_scan(
  files_to_scan: [{ relativeFilePath: "relative/path.ts", content: "..." }],
  repository_name: "repo-name"
)
```

---

## Mode: Directory Scan

### Step 1: List Directory Contents

List all code files in the specified directory recursively.

**Include:** `.ts`, `.tsx`, `.js`, `.jsx`, `.py`, `.go`, `.java`, `.rb`, `.php`, `.cs`

**Exclude:**
- `node_modules/`, `dist/`, `build/`, `.next/`
- Test files: `*.spec.ts`, `*.test.ts` (unless explicitly requested)
- Lock files and configs

### Step 2: Batch and Scan

If more than 50 files, split into batches. Read content and send to Aikido.

---

## Mode: Git Changes Scan

### Step 1: Get Modified Files

Run git commands to identify changed files:

```bash
# Unstaged changes
git diff --name-only

# Staged changes
git diff --staged --name-only

# Both
git diff --name-only HEAD
```

### Step 2: Filter Code Files

Apply the same include/exclude filters as Directory Scan.

### Step 3: Read and Scan

Read current content of each modified file (not the diff, the full file) and send to Aikido.

---

## Report Format

Present findings consistently across all modes:

```markdown
## 🔒 Security Scan Results

**Mode:** {File Scan | Directory Scan | Git Changes Scan}
**Files scanned:** {count}
**Vulnerabilities found:** {count}
**Secrets detected:** {count}

---

### Findings

#### 🔴 Critical
- **{file}:{line}** — {title}
  - {remediation}

#### 🟠 High
- **{file}:{line}** — {title}
  - {remediation}

#### 🟡 Medium
- **{file}:{line}** — {title}
  - {remediation}

#### 🟢 Low
- **{file}:{line}** — {title}
  - {remediation}

---

✅ **No issues found** — Scanned files are clean.
```

---

## False Positive Evaluation

Same rules as PR scan:

| Finding Type | Verify Before Reporting |
|--------------|------------------------|
| NoSQL Injection | Check for Zod/Joi validation upstream |
| Hardcoded Secret | Check if it's a placeholder or test value |
| Path Traversal | Check if path uses constants, not user input |

Mark likely false positives with explanation:

```
⚠️ [LIKELY FALSE POSITIVE] {title} in {file}:{line}
   Reason: {evidence from code}
```
