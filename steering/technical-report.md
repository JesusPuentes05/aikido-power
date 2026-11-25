# Technical Report Workflow

Generate a technical security report to deliver to the client's technical team. It covers one repository by default, or several repositories in a single document when the user asks for it. It lists the state per repository, the breakdown by finding type, the trend from saved snapshots, the detailed findings and the recommended priorities.

## Audience and Use

| | |
|---|---|
| **Audience** | The client's technical team (developers, DevOps, security). It can also be used by your own engineering team. |
| **Purpose** | Technical deliverable to plan remediation: what is open, where it is and how to fix it. |
| **Contains** | Counts and score per repository, breakdown by type, trend, detailed findings with file, package, CVE and remediation, recommended priorities. |
| **Not for** | Non-technical readers such as the client's managers: use `client-report.md` for them. Not for personal score tracking: use `score-tracking-report.md`. |

If the user only says "security report", "technical report" or "reporte técnico" without more context, use this flow. If the user only says "executive report" or "reporte ejecutivo", ask which one they want: this technical report, the client report, or the score tracking report.

The report says where vulnerabilities and exposed secrets are, because the technical team needs that to fix them. That makes it sensitive: **never include secret values** (only the type of secret and the file), and deliver it only to the party that owns the repository, through a controlled channel. Do not publish it or put it in a repository with wide access.

If the report covers repositories that belong to different clients, generate one report per client. Never mix them in one document.

---

## Step 1: Collect Parameters

Ask for everything that is missing in a single message. Do not ask for what the user already said.

| Parameter | Rule |
|-----------|------|
| **Project name** | Always ask. It goes in the title. Suggest the repository name as a default. Never reuse a name from a previous report without asking. |
| **Document version** | Always ask (for example `v1.0.0`). Do not assume or increment it. If earlier technical reports exist in `security/reports/`, mention the latest version found as a reference only. |
| **Language** | Always ask: Spanish or English. Do not assume. |
| **Repositories** | Use only the repositories the user names. If the Kiro workspace has several candidate repositories, suggest them and wait for confirmation. Never add repositories on your own. |
| **Role of each repository** | Only when there is more than one repository. Ask, or infer from the name and confirm: `backend`, `frontend`, or another label. Aikido does not provide this. |
| **Period** | Optional, for the trend. If not given, use all available snapshots. |
| **Rows per table** | Optional. Default is 10. |

If you cannot confirm a repository name, validate the exact name with `aikido_workspaces_list` and `aikido_issues_list` before continuing. If the user has no access to it, stop and say so.

---

## Step 2: Fetch Current Issues

For each repository:

```
aikido_issues_list(repo_name: "repo-name", page: 0)
```

**Fetch every page.** Keep requesting `page: 1, 2, 3...` until a page returns no issues. A report built from only the first page has wrong totals.

Do not filter by type or severity, so the totals match Aikido. Ignored and snoozed issues are not in the feed and are not counted.

Each issue includes `location.branch_name`. If the issues of one repository come from more than one branch, ask which branch to report (use `repo_branch_name`) and never mix branches. State the branch in the report header.

Fields used:

| Report column | Field |
|---------------|-------|
| Severity | `issue_severity_label` and `issue_severity` |
| Finding | `issue_title` |
| File | `issue_file` (and the line when the feed returns it) |
| Package | `issue_affected_package` |
| CVE / ID | `issue_cve_id` (fall back to the title if empty) |
| Installed version | `issue_installed_version` |
| Remediation | `issue_remediation` (fall back to `issue_patched_versions`) |
| Type | `issue_type` |

**Severity labels:** use the label Aikido returns (`critical`, `high`, `medium`, `low`). Do not derive it from the numeric score. An issue without a label is reported as "unclassified" and the user is told.

---

## Step 3: Classify by Type

| Report type | Aikido `issue_type` |
|-------------|---------------------|
| Dependencias / Dependencies | `open_source` |
| Secretos filtrados / Leaked secrets | `leaked_secret` |
| SAST (código / code) | `sast`, `ai_code_analysis` |
| IaC | `iac` |
| Otros / Other | everything else (`cloud`, `malware`, `eol`, `license`, `docker_container`, `mobile`, `scm_security`, and so on) |

Always show the four main rows, even with 0. Show "Otros / Other" only when it has issues. Type counts must add up to the repository total.

---

## Step 4: Snapshots and Trend

1. Ask the user whether to save today's snapshot first, using steps 3 and 4 of `security-report.md`. Saving writes a file, so do not do it without a yes. A saved snapshot makes the last row of the trend match the detail of the report.
2. Load `security/reports/YYYY-MM-DD-{repo-name}.json` for each repository, filtered to the period. Sort by date ascending.
3. **Recalculate the score of every snapshot from its severity counts.** Ignore any stored `security_score`.

```
score = 100 - (critical * 3) - (high * 2) - (medium * 1), minimum 0
```

Per repository:

- **Snapshots exist:** build the trend table. If today's snapshot was not saved, add a last row with today's date and the label "estado actual (no guardado)".
- **No snapshots:** do not invent a trend. Show the baseline text in the template.

Calculate the progress block between the last two rows of the table.

---

## Step 5: Generate the Report

Save to:

```
security/reports/technical-report-{project-slug}-v{version}-{YYYY-MM-DD}.md
```

Write the report in the language the user chose. The template is in Spanish. For English, translate the fixed text faithfully and keep the same structure. Use `🔴 Crítico`, `🟠 Alto`, `🟡 Medio`, `🟢 Bajo` for severity (`Critical`, `High`, `Medium`, `Low` in English), followed by the score in parentheses, for example `🟠 Alto (82)`.

### Row limits and ordering

- Sort every detail table by severity score, highest first. Break ties by name, so the order is stable.
- Show at most the rows-per-table limit (default 10). After the table, add one sentence that summarizes the rest with counts (for example "Los 15 hallazgos restantes corresponden a 9 paquetes de severidad media y baja.").
- **Leaked secrets:** group by file and secret type. Show the highest severity and the number of detections. Never show values.
- **Dependencies:** group by package, one row per package, not one row per CVE. A single package can account for most of the findings, and one row per CVE would fill the table with the same package. For each package show: the highest severity among its findings, the number of findings, the installed version, the target version and up to 3 example CVE or Aikido IDs (the most severe first). The target version is the highest version that Aikido recommends among the package's findings (read it from `issue_remediation`; if it has no version, use the highest of `issue_patched_versions`). Updating to it covers every finding of that package. If a package appears with more than one installed version, list each one.
- If one package accounts for more than half of the dependency findings, say so in one sentence after the table.
- **End-of-life conflict:** if a package in the table also has an `eol` finding (for example a framework version that no longer receives security updates), the recommended target version is the one that resolves the end-of-life finding, as long as it is equal or higher than the table's target. Mention it in the priorities and note that it also covers the dependency findings of that package. Do not recommend two different targets for the same package.
- **Subsection numbers** are consecutive. Skip the types that have no issues and renumber the rest (for example, without leaked secrets, SAST becomes 4.1).

### Template (Spanish)

````markdown
# Reporte de seguridad — {Project name}

**Versión de documento:** {version}
**Fecha:** {today}
**Repositorios:** {repo} ({role}, rama {branch}), ...

---

## 1. Resumen ejecutivo

Este reporte consolida el estado de seguridad de {the repository | los {n} repositorios} de {Project name} a partir del feed de issues de Aikido. Incluye un panorama por severidad y tipo, {la tendencia de {repos with snapshots} contra snapshots históricos, }y el listado detallado de hallazgos con su remediación.

| Repositorio | Issues abiertos | Críticos | Altos | Medios | Bajos | Security Score |
|-------------|-----------------|----------|-------|--------|-------|----------------|
| {repo} ({role}) | {n} | {n} | {n} | {n} | {n} | {score}/100 |
| **Total** | {n} | {n} | {n} | {n} | {n} | — |

Show the Total row only when there is more than one repository. Never calculate a combined score.

**Lectura rápida:**

Include each bullet only when its condition is true.
- {critical total = 0}: No hay issues críticos abiertos en {el repositorio | ninguno de los repositorios}.
- {critical total > 0}: Hay {n} issues críticos abiertos, en {repos}.
- {more than one repository}: {repo} concentra el {pct}% de los hallazgos ({n} de {total}), principalmente por {the two types with most issues in that repo}.
- {leaked secrets > 0}: El riesgo más accionable son los {n} secretos filtrados de {repo} (en archivos versionados).
- {for each repo with high or critical issues}: En {repo}, los hallazgos altos corresponden a {types, and up to 3 package names or file names}.
- {for each repo with 0 high and 0 critical}: {repo} no tiene hallazgos altos ni críticos.

Round percentages to the nearest whole number.

---

## 2. Desglose por tipo de hallazgo

| Tipo | {repo or role 1} | {repo or role 2} | Total |
|------|------------------|------------------|-------|
| Dependencias open source | {n} | {n} | {n} |
| Secretos filtrados | {n} | {n} | {n} |
| SAST (código) | {n} | {n} | {n} |
| IaC | {n} | {n} | {n} |
| Otros | {n} | {n} | {n} |
| **Total** | {n} | {n} | {n} |

One column per repository. Omit the Total column when there is one repository.

---

## 3. Tendencia

Repeat this block for each repository, with the heading "3.{i} Tendencia de {repo} ({role})".

{With snapshots:}
Snapshots registrados en `security/reports/`:

| Fecha | Total | Críticos | Altos | Medios | Bajos | Score |
|-------|-------|----------|-------|--------|-------|-------|
| {date} | {n} | {n} | {n} | {n} | {n} | {score} |

**Nota metodológica:** Aikido puede ajustar sus criterios de clasificación de severidad con el tiempo. Parte de la variación en críticos y altos puede deberse a esos ajustes y no solo a remediaciones. {Optional text added by the user.}

**Progreso {date A} → {date B}:**
- Total de issues: {a} → {b} ({+/-n})
- Críticos: {a} → {b} ({+/-n})
- Score: {a} → {b} ({+/-n})

{No snapshots:}
No hay snapshots previos de {repo}. Esta es la primera medición registrada y sirve como línea base. Las próximas ejecuciones permitirán mostrar la tendencia.

Show the nota metodológica only when there are two or more rows.

---

## 4. Detalle de hallazgos — {Role} ({repo})

Repeat one section per repository (4, 5, ...). Include a subsection only when that type has issues.

### 4.1 Secretos filtrados ({n}) — prioridad de atención

Se detectaron claves y tokens en archivos versionados. Por su naturaleza conviene tratarlos primero: rotar los secretos expuestos, sacarlos del repositorio e inyectarlos mediante variables de entorno o un gestor de secretos.

| Severidad | Hallazgo | Archivo | Detecciones |
|-----------|----------|---------|-------------|
| {label} ({score}) | {secret type} | {file} | {n} |

### 4.2 Vulnerabilidades SAST en código ({n})

| Severidad | Hallazgo | Archivo (línea) | Remediación |
|-----------|----------|-----------------|-------------|
| {label} ({score}) | {title} | {file} ({line}) | {remediation} |

{Sentence about the rows not shown.}

### 4.3 Dependencias open source vulnerables ({n} hallazgos en {p} paquetes)

| Severidad máxima | Paquete | Hallazgos | Versión instalada | Actualizar a | Ejemplos de CVE / ID |
|------------------|---------|-----------|-------------------|--------------|----------------------|
| {label} ({score}) | {package} | {n} | {installed} | {target} | {up to 3 ids} |

{Sentence about the packages not shown, and the sentence about a dominant package when it applies.}

### 4.4 Otros hallazgos ({n})

Same table as 4.2, with a Tipo column.

---

## {last number}. Prioridades recomendadas

Numbered list, in this order. Include only the items that apply and number them consecutively. Take counts, packages and files from the data.

1. {critical issues of any type}: Atender los {n} issues críticos de {repo}: {types and up to 3 names}.
2. {leaked secrets}: Rotar y remover los secretos de {repo}. Son {n} detecciones en {file kinds, e.g. archivos de entorno, archivos de configuración, scripts, documentación}. Rotar las credenciales expuestas, sacarlas del repositorio e inyectarlas por entorno o un gestor de secretos.
3. {SAST with high severity}: Corregir los hallazgos SAST altos de {repo}: {types}. Centralizar la validación de entrada en los puntos señalados.
4. {dependencies with high severity}: Actualizar las dependencias de severidad alta de {repo}: {up to 5 packages with the target version from the table}. {If one package dominates: say how many findings that update covers.}
5. {remaining dependencies}: Ronda de actualización de dependencias de {repo}: {up to 8 package names with target version}.
6. {SAST, IaC or other findings not covered above}: Revisar los {n} hallazgos restantes de {repo} ({types and up to 3 short descriptions}). Son de menor prioridad, pero tienen remediación sencilla.
````

If an English report is requested, translate the template text and keep every number and name unchanged.

---

## Step 6: Present to the User

After saving the file:
1. Show the full report in the chat so it can be copied.
2. Give the path where it was saved.
3. State the source of the numbers: the date of the Aikido query, the branch, and the snapshot dates used.
4. Remind the user that the report says where vulnerabilities and secrets are: deliver it only to the client that owns the repository, through a controlled channel, and do not commit it to a repository with wide access. Do not commit it unless the user asks.
5. If there were no snapshots and the user did not save today's, offer once to save a baseline for the next report.

---

## Scope and Safety

- Only report repositories the user named in this conversation. If a name is ambiguous, ask.
- This flow only reads data from Aikido. Never call `aikido_ignore_issue`, `aikido_snooze_issue`, `aikido_adjust_issue_severity` or their undo counterparts from here.
- The only file it writes, besides the report, is the optional snapshot, and only after a yes from the user.
- If Aikido returns an authentication error, ask the user to run "Login to Aikido" and retry.
