# Aikido Security Scanner — Kiro Power

Power de seguridad para [Kiro](https://kiro.dev) que integra el servidor MCP de [Aikido](https://www.aikido.dev/) para ejecutar análisis SAST y detección de secretos directamente desde el IDE.

## ¿Qué hace?

- Escanea archivos de código en busca de vulnerabilidades SAST y secretos hardcodeados.
- Permite revisar la seguridad de un PR antes de hacer merge.
- Gestiona y prioriza los issues de seguridad de un repositorio.
- Genera reportes de seguridad con métricas cuantificables para seguimiento de cumplimiento.
- Produce reportes ejecutivos listos para copiar en documentos de evaluación.

## Requisitos

- Node.js 18+ con `npx` disponible
- Una cuenta de Aikido (solicítala a tu tech lead)

## Instalación

1. Instala el Power desde el panel de Powers en Kiro.
2. La primera vez que ejecutes un comando de Aikido, Kiro te proporcionará un enlace de autenticación. Ábrelo en tu navegador y autoriza el acceso.
3. Listo — la sesión persiste entre reinicios de Kiro.

## Estructura del proyecto

```
aikido-security-scanner/
├── POWER.md                        # Documentación principal del power
├── mcp.json                        # Configuración del servidor MCP
├── README.md                       # Este archivo
└── steering/
    ├── executive-report.md         # Guía para reportes ejecutivos
    ├── file-scan.md                # Guía para escaneo de archivos
    ├── issues-triage.md            # Guía para triaje de issues
    ├── pr-scan.md                  # Guía para escaneo de PRs
    └── security-report.md          # Guía para reportes de seguridad
```

## Uso

### Escaneo rápido de archivos

> "Scan src/controllers/auth.controller.ts for security issues"

### Revisión de seguridad en PRs

> "Scan PR #45 for security vulnerabilities"

### Triaje de issues

> "Show me all open Aikido issues for mi-repositorio"

### Reporte de seguridad

> "Generate a security report for mi-repositorio"

### Reporte ejecutivo

> "Generate the executive security report for mi-repositorio"

## Autenticación

El power usa el flujo de login por navegador de Aikido. Cada miembro del equipo se autentica con su propia cuenta la primera vez. Si el token expira, simplemente pide a Kiro: *"Login to Aikido"*.

## Buenas prácticas

- Ejecuta escaneos antes de crear PRs para detectar issues temprano.
- Genera snapshots de seguridad al menos una vez al mes.
- Al ignorar un issue, documenta siempre la justificación.
- Revisa inmediatamente los hallazgos de severidad crítica y alta.

## Solución de problemas

| Problema | Solución |
|----------|----------|
| El servidor MCP no inicia | Verifica Node.js 18+ con `node --version`. Prueba `npx -y @aikidosec/mcp` manualmente. |
| Errores de autenticación | Regenera tu token en el dashboard de Aikido. |
| Escaneo sin resultados | Verifica que el archivo sea de un lenguaje soportado (.ts, .js, .py, .go, etc.) |

## Tecnologías

- **MCP Server:** `@aikidosec/mcp`
- **Plataforma:** [Kiro](https://kiro.dev)
- **Motor de análisis:** [Aikido Security](https://www.aikido.dev/)

## Autor

Codster
