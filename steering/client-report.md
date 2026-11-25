# Client Report Workflow

Generate a plain-language security report to deliver to a client. It covers one repository by default, or several repositories in a single document when the user asks for it. It explains what was found, what it means and what to do, without technical detail.

## Audience and Use

| | |
|---|---|
| **Audience** | The client and their managers. The document leaves the company. |
| **Purpose** | Deliverable that explains the security status of the project in plain language. |
| **Contains** | Explanation of Aikido, executive summary, glossary, interpretation of results, evolution. Counts and fixed text only. |
| **Not for** | Personal or internal tracking. It has no score, no file paths, no issue titles. Use `score-tracking-report.md` for that. |

If the user only says "executive report" or "reporte ejecutivo" without more context, ask which one they want: this client report, or the score tracking report.
---

## Trigger

User asks for a security report to give to a client or a non-technical audience, for example:
- "Generate the client report for the backend"
- "Genera el reporte de seguridad para el cliente de my-api"
- "I need a security report for the client covering backend and frontend"
- "Reporte de seguridad para entregar al cliente"

If the user asks for their own score, a timeline or the top risks with files, use `score-tracking-report.md` instead.

---

## Step 1: Collect Parameters

Ask for everything that is missing in a single message. Do not ask for what the user already said.

| Parameter | Rule |
|-----------|------|
| **Project name** | Always required. It goes in the title (e.g., "Reporte de Seguridad — {Project name}"). Suggest the repository name as a default. Never reuse a name from a previous report without asking. |
| **Repositories** | Use only the repositories the user names. If the Kiro workspace contains several candidate repositories, suggest them and wait for confirmation. Never add repositories on your own. |
| **Role of each repository** | Only when there is more than one repository. Ask or infer from the name and confirm: `backend`, `frontend`, or another label. Aikido does not provide this data. |
| **Language** | Always ask: Spanish or English. Do not assume. |
| **Period** | Optional, used for the evolution section. If not given, use all available snapshots. |

If the user names a repository that is not in the current Kiro workspace or that you cannot confirm, validate the exact name with `aikido_workspaces_list` and `aikido_issues_list` before continuing. If the user has no access to it, stop and say so.

---

## Step 2: Fetch Current Issues

For each repository:

```
aikido_issues_list(repo_name: "repo-name", page: 0)
```

**Fetch every page.** Keep requesting `page: 1, 2, 3...` until a page returns no issues. A report built from only the first page has wrong totals.

Do not filter by issue type or severity, so the totals match what Aikido shows. Ignored and snoozed issues do not appear in the feed, so they are not counted.

Record, per repository:
- total open issues
- count by severity label (`critical`, `high`, `medium`, `low`)
- count by category (see Step 3)

---

## Step 3: Classify by Category

| Category (report) | Aikido `issue_type` |
|-------------------|---------------------|
| Credentials | `leaked_secret` |
| Dependencies | `open_source` |
| Code | `sast`, `ai_code_analysis` |
| Other | everything else (`iac`, `cloud`, `malware`, `eol`, `license`, `docker_container`, `mobile`, `scm_security`, and so on) |

The category counts must add up to the repository total. Show the "Other" row whenever it has issues so no finding is lost.

---

## Step 4: Load Snapshots (Evolution)

Look in `security/reports/` for `YYYY-MM-DD-{repo-name}.json`, filtered to the requested period.

- **Snapshots exist:** the start point is the earliest snapshot in the period. The end point is the current state from Step 2, not the latest snapshot.
- **No snapshots:** this is a **baseline**. Do not invent a previous value.

If there are no snapshots, offer once at the end to save one with the `security-report.md` flow, so the next report can show evolution. Do not save it without asking.

For several repositories, calculate evolution per repository. Show a combined total only when every repository has a start snapshot.

---

## Step 5: Generate the Report

Save to:

```
security/reports/client-report-{project-slug}-{YYYY-MM-DD}.md
```

Write the report in the language the user chose. The template below is in Spanish. For English, translate the fixed text faithfully and keep the same structure.

### Data rules

- Use only counts and the fixed text below.
- **Never include** secret values, file paths, issue titles, code, internal URLs or issue IDs. The document is pasted into shared documents.
- Every number comes from Aikido or from a snapshot. Never estimate.
- Use the conditional wording as indicated. Do not say "no critical issues" unless the critical count is 0.

### Template (Spanish)

````markdown
# Reporte de Seguridad — {Project name}

**Fecha:** {today}
**Repositorios analizados:** {list of repo names}

---

## 1. ¿Qué es Aikido?

Aikido es una herramienta que revisa los proyectos de software para identificar posibles problemas de seguridad. Permite detectar accesos que podrían estar expuestos, componentes que necesitan actualizarse y errores que podrían comprometer el funcionamiento o la seguridad de los sistemas.

Los resultados ayudan a identificar qué problemas existen, cuáles requieren mayor atención y qué acciones se recomiendan para solucionarlos.

---

## 2. Resumen ejecutivo

Se realizó una revisión de seguridad de los proyectos de {Project name}, identificando {total} hallazgos pendientes de atención.

{Single repository:}
- {n} hallazgos en {repo role description}.
{Several repositories, one line each:}
- {n} hallazgos en los servicios internos (backend).
- {n} hallazgos en la aplicación utilizada por los usuarios (frontend).

{If critical = 0:}
- 0 hallazgos críticos en la evaluación actual.

Aunque no se identificaron problemas clasificados como críticos, existen riesgos que deben atenderse, principalmente relacionados con {list only the categories that have issues, e.g. "accesos potencialmente expuestos, seguridad del código y actualización de componentes"}.

{If critical > 0, replace the two paragraphs above with:}
- {critical} hallazgos críticos en la evaluación actual.

Se identificaron {critical} problemas clasificados como críticos que requieren atención prioritaria, además de otros riesgos relacionados con {categories with issues}.

---

## 3. Glosario

| Concepto | ¿Qué significa? |
|----------|-----------------|
| Backend | Parte del sistema que procesa y administra la información. |
| Frontend | Parte visual del sistema con la que interactúan los usuarios. |
| Hallazgo | Posible problema detectado durante la revisión de seguridad. |
| Credenciales | Llaves o códigos que permiten acceder a sistemas y servicios. |
| Vulnerabilidad | Debilidad que podría comprometer la seguridad del sistema. |
| Dependencias | Componentes externos que se utilizan para desarrollar y operar el software. |

Include the Backend and Frontend rows only if the report uses those roles. Include the Credenciales and Dependencias rows only if those categories have issues.

---

## 4. Interpretación de resultados

| Estado actual | ¿Qué significa? | Acciones recomendadas |
|---------------|-----------------|-----------------------|
| {total} hallazgos pendientes, {critical_phrase}. | Los proyectos presentan aspectos de seguridad que necesitan atención{if critical = 0: ", aunque ninguno está clasificado en el nivel de mayor gravedad"}. | Revisar y atenderlos según su importancia. |
| {n} posibles credenciales expuestas. | Se encontraron llaves de acceso en archivos del proyecto. Si siguen activas, podrían permitir accesos no autorizados. | Revisar y reemplazar las credenciales expuestas y proteger su almacenamiento. |
| {n} hallazgos en componentes externos. | Algunos componentes utilizados por los proyectos presentan fallas de seguridad conocidas. | Actualizar los componentes afectados y comprobar su funcionamiento. |
| {n} hallazgos relacionados con el código. | Se identificaron partes del sistema que podrían permitir comportamientos no deseados si reciben información manipulada. | Corregir los puntos identificados y realizar pruebas de seguridad. |
| {n} hallazgos de otro tipo. | Se detectaron riesgos en otras áreas, como infraestructura o configuración. | Revisar cada caso con el equipo responsable. |

`{critical_phrase}` is "ninguno crítico" when critical = 0, otherwise "{critical} críticos".
Include a row only when its count is greater than 0. If the total is 0, replace the table with: "No se encontraron hallazgos pendientes en la evaluación actual."

When there are several repositories, add a breakdown table:

| Repositorio | Rol | Total | Críticos | Altos | Medios | Bajos |
|-------------|-----|-------|----------|-------|--------|-------|
| {repo} | {role} | {n} | {n} | {n} | {n} | {n} |

---

## 5. Evolución de la seguridad

{Baseline, no snapshots:}
Esta es la primera medición registrada para {repo names}. Las próximas revisiones permitirán comparar los resultados y mostrar la evolución.

{With snapshots, one paragraph per repository:}
En el proyecto {role or repo name}, los hallazgos pendientes {disminuyeron | aumentaron} de {start} a {end} entre {start date} y {end date}{, lo que representa una {reducción | un aumento} aproximada del {pct}%}.

{No change:}
En el proyecto {role or repo name}, los hallazgos pendientes se mantuvieron en {n} entre {start date} y {end date}.

Add the percentage only when start > 0. Round to the nearest whole number.

Si todos los repositorios tienen punto de partida, agrega:

| Repositorio | {start date} | {end date} | Cambio |
|-------------|--------------|------------|--------|
| {repo} | {n} | {n} | {+/-n} |
| **Total** | {n} | {n} | {+/-n} |

{Fixed note, shown whenever there is a comparison:}
Esta variación refleja una mejora en los resultados de la revisión. Sin embargo, parte de los cambios también puede deberse a ajustes en los criterios de evaluación de Aikido.

If the count went up, replace "refleja una mejora" with "refleja un cambio".
````

---

## Step 6: Present to the User

After saving the file:
1. Show the full report in the chat so it can be copied right away.
2. Give the path where it was saved.
3. State briefly where the numbers come from: the date of the Aikido query and, if used, the snapshot dates.
4. If there were no snapshots, make the offer from Step 4.

---

## Scope and Safety

- Only report repositories the user named in this conversation. If a name is ambiguous, ask.
- This flow only reads data. Never call `aikido_ignore_issue`, `aikido_snooze_issue` or `aikido_adjust_issue_severity` from here.
- Snapshots in `security/reports/` contain issue titles and file paths. Remind the user not to publish that folder in a repository with wide access.
- If Aikido returns an authentication error, ask the user to run "Login to Aikido" and retry.
