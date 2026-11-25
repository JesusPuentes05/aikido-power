# Aikido Security Scanner (Kiro Power)

Un [Power de Kiro](https://kiro.dev) que conecta Kiro con [Aikido Security](https://www.aikido.dev). Permite escanear código en busca de vulnerabilidades SAST y secretos hardcodeados, gestionar los issues de Aikido y generar reportes de seguridad con métricas para revisiones de cumplimiento o evaluaciones de desempeño.

## Qué hace

| Flujo | Archivo de steering | Descripción | Ejemplo de mensaje |
|-------|---------------------|-------------|--------------------|
| Escaneo de archivos | `steering/file-scan.md` | Escanea archivos concretos, un directorio o tus cambios locales de git (staged y unstaged). | "Escanea src/controllers/auth.controller.ts en busca de problemas de seguridad" |
| Escaneo de PR | `steering/pr-scan.md` | Escanea cada archivo de código modificado en un pull request. Opcionalmente publica los resultados como comentario en el PR. | "Escanea el PR #45 en busca de vulnerabilidades" |
| Triage de issues | `steering/issues-triage.md` | Lista los issues abiertos de Aikido de un repositorio, revisa un issue concreto o ignora uno con una razón documentada. | "Muéstrame todos los issues abiertos de Aikido de my-backend" |
| Reporte de seguridad | `steering/security-report.md` | Guarda un snapshot JSON de los issues actuales y lo compara con el anterior (issues resueltos, nuevos y persistentes, y un score de seguridad). | "Genera un reporte de seguridad de my-api" |
| Reporte de score (interno) | `steering/score-tracking-report.md` | Sigue tu score de seguridad en el tiempo como métrica de evaluación, a partir de los snapshots guardados. Incluye archivos y títulos de issues, así que no es para clientes. | "Genera el reporte de score de my-api" |
| Reporte para el cliente (externo) | `steering/client-report.md` | Reporte en lenguaje sencillo para entregar al cliente, en español o inglés. Cubre un repositorio por defecto, o varios en un solo documento. Pregunta el nombre del proyecto y el idioma. Sin score, sin rutas ni títulos. Muestra la evolución con snapshots, o un baseline si no los hay. | "Genera el reporte para el cliente de my-backend" |

Los flujos de escaneo también señalan posibles falsos positivos, como entradas ya validadas con Zod/Joi, secretos de ejemplo o rutas construidas con constantes.

## Estructura del repositorio

```
aikido-power/
├── POWER.md      Definición del Power, onboarding y troubleshooting
├── mcp.json      Configuración del servidor MCP (@aikidosec/mcp)
├── README.md     Este archivo
└── steering/     Una guía de flujo por cada capacidad
```

## Requisitos

- **Kiro** con soporte para Powers.
- **Node.js 18 o superior**, con `npx` disponible. El servidor MCP se ejecuta como `npx -y @aikidosec/mcp`.
- **Una cuenta de Aikido** con acceso a los repositorios que quieras escanear.
- **Acceso a red** hacia Aikido y hacia el registro de npm.
- **Opcional, para escaneo de PR:** un servidor MCP de GitHub o herramientas equivalentes configuradas en Kiro. Este Power no instala ninguno. Sin él, el escaneo de PR no puede obtener la lista de archivos modificados.
- **Opcional, para reportes:** un repositorio git donde se pueda crear y versionar `security/reports/`.

## Instalación

1. Abre el panel de Powers en Kiro e instala este Power desde este repositorio (ruta local o URL de Git).
2. Comprueba que Node.js está disponible:
   ```
   node --version
   ```
3. Pídele a Kiro cualquier comando de Aikido, por ejemplo "Escanea src/app.ts en busca de problemas de seguridad".

## Autenticación

En el primer uso, Kiro muestra un enlace de inicio de sesión. Ábrelo en el navegador y autoriza el acceso. La sesión se guarda localmente. Para iniciar sesión de nuevo, dile a Kiro: "Login to Aikido".

Cada persona se autentica con su propia cuenta de Aikido, y ve los repositorios y workspaces a los que esa cuenta tenga acceso. El Power no limita el acceso por su cuenta. Los permisos se controlan en Aikido.

Si el servidor MCP informa un error de token inválido o no autorizado, comprueba si tu configuración de Aikido exige una API key. En ese caso define la variable de entorno `AIKIDO_API_KEY` y reinicia Kiro. No subas la clave al repositorio.

## Ejemplos de uso

```
Escanea src/controllers/auth.controller.ts en busca de problemas de seguridad
Revisa mis cambios staged en busca de vulnerabilidades
Escanea el PR #45 en busca de vulnerabilidades
Muéstrame todos los issues abiertos de Aikido de my-backend
Ignora el issue 12345, es un falso positivo validado por un schema
Genera un reporte de seguridad de my-api
Genera el reporte de score de my-api
Genera el reporte para el cliente de my-backend
```

Los flujos están escritos en inglés, pero el agente entiende y responde en el idioma que uses. El reporte para el cliente pregunta en qué idioma se debe generar el documento.

## Reporte para el cliente

Es el flujo pensado para entregar al cliente. Funciona así:

- Pregunta el **nombre del proyecto** para el título y el **idioma** del documento (español o inglés).
- Usa solo los repositorios que nombres. Si nombras varios (por ejemplo backend y frontend), los une en un solo documento con un desglose por repositorio.
- Obtiene todos los issues abiertos desde Aikido y los agrupa en credenciales, dependencias, código y otros.
- Calcula la evolución con los snapshots de `security/reports/`. Si no hay ninguno, el reporte sale como baseline y ofrece guardar un snapshot para la próxima vez.
- No muestra score. Tampoco incluye valores de secretos, rutas de archivos, títulos de issues ni IDs. Solo conteos y texto explicativo.
- Es de solo lectura. No ignora, pospone ni cambia la severidad de ningún issue.

Los datos históricos solo existen si antes se guardaron snapshots. Aikido devuelve únicamente el estado actual.

## Datos de los reportes

Los archivos se escriben en `security/reports/`, en la raíz del workspace:

- `YYYY-MM-DD-{repo-name}.json`: un snapshot por día. Volver a generar el reporte el mismo día lo sobrescribe.
- `score-tracking-report-{period}-{repo-name}.md`: el reporte de score, de uso interno.
- `client-report-{project}-{YYYY-MM-DD}.md`: el reporte para el cliente, en lenguaje sencillo.

El score de seguridad es `100 - (3 × críticos) - (2 × altos) - (1 × medios)`, con mínimo 0. Los issues de severidad baja no lo afectan. Lo usan el reporte de seguridad y el reporte de score. El reporte para el cliente no lo muestra.

Versiona la carpeta `security/reports/` para que el equipo comparta el historial. Los snapshots contienen títulos de issues y rutas de archivos, así que comprueba que la visibilidad de tu repositorio sea adecuada.

## Troubleshooting

| Problema | Qué probar |
|----------|------------|
| No aparecen las herramientas de Aikido | Ejecuta `node --version` (se necesita 18+) y luego `npx -y @aikidosec/mcp` manualmente para ver el error. Reinicia Kiro. |
| "Unauthorized" o "sign-in required" | Dile a Kiro "Login to Aikido". Si falla, revisa `AIKIDO_API_KEY` por espacios de más o un token vencido. |
| El escaneo no devuelve nada para un archivo vulnerable | Comprueba que el lenguaje sea compatible, que el nombre del repositorio coincida con el de Aikido y que el archivo no esté vacío. |
| El escaneo de PR no puede listar archivos | Configura un servidor MCP de GitHub en Kiro. |
| Los totales del reporte para el cliente parecen bajos | Comprueba que el nombre del repositorio sea exactamente el de Aikido. El flujo recorre todas las páginas de resultados. |

## Notas

- Ignorar un issue en Aikido lo quita de los escaneos y snapshots futuros, y el reporte lo cuenta como resuelto. Da siempre una razón documentada.
- El reporte de score incluye las secciones "Actions Taken" y "Next Steps" con texto de ejemplo. Edítalas antes de entregarlo.
- Si pides solo "el reporte ejecutivo", el agente te pregunta si quieres el reporte de score (interno) o el reporte para el cliente.
