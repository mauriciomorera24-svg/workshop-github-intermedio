# ⚙️ Workshop: GitHub Intermedio

## Pipelines, Políticas y Productividad con .NET 9

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-Workflows-2088FF?logo=githubactions&logoColor=white)
![.NET](https://img.shields.io/badge/.NET%209-SDK-512BD4?logo=dotnet&logoColor=white)
![GitHub CLI](https://img.shields.io/badge/GitHub%20CLI-Productivity-181717?logo=github&logoColor=white)
![NuGet](https://img.shields.io/badge/NuGet-Packages-004880?logo=nuget&logoColor=white)
![Nivel](https://img.shields.io/badge/Nivel-Intermedio-orange)
![Duración](https://img.shields.io/badge/Duración-2%20horas-blue)

---

## 📋 Tabla de Contenidos

- [Introducción](#-introducción)
- [¿Qué vas a aprender?](#-qué-vas-a-aprender)
- [Pre-requisitos](#-pre-requisitos)
- [Preparación del entorno](#-preparación-del-entorno)
- [Arquitectura del proyecto base](#-arquitectura-del-proyecto-base)
- [Agenda del Workshop](#-agenda-del-workshop)
- [Módulo 1: GitHub Actions](#-módulo-1-github-actions-60-min)
- [Módulo 2: Políticas de repositorio](#-módulo-2-políticas-de-repositorio-codeowners-y-merge-25-min)
- [Módulo 3: Tags, Releases y Packages](#-módulo-3-tags-releases-y-github-packages-25-min)
- [Checklist final](#-checklist-final)
- [Troubleshooting](#-troubleshooting)
- [Recursos adicionales](#-recursos-adicionales)
- [FAQ](#-preguntas-frecuentes)
- [Créditos](#-créditos)

---

## 🎯 Introducción

Este workshop práctico de 2 horas te llevará del conocimiento básico de Git y GitHub a un dominio real de las herramientas que hacen la diferencia en equipos profesionales. Aprenderás a:

- ✅ Diseñar pipelines CI con jobs encadenados, outputs y artefactos
- ✅ Usar matrix strategy para probar contra múltiples versiones y sistemas operativos
- ✅ Diagnosticar y corregir workflows rotos (un skill que se usa a diario)
- ✅ Configurar branch protection, CODEOWNERS y políticas de merge
- ✅ Crear tags con versionamiento semántico y GitHub Releases
- ✅ Publicar paquetes NuGet a GitHub Packages desde un workflow

> 💡 **Filosofía del workshop:** No se trata de memorizar sintaxis YAML. Se trata de entender *por qué* se estructura un pipeline de cierta forma, *qué problema resuelve* cada política, y *cuándo* conviene cada herramienta. La aplicación base es real, no un "hello world".

### Escenario: FinancialUtils 🏦

La aplicación base es una librería de utilidades financieras en .NET 9 con operaciones de interés compuesto, amortización de préstamos y formateo de valores monetarios. Es lo suficientemente real para que los workflows tengan algo con qué trabajar, sin ser compleja al punto de distraer del objetivo.

| Aspecto | Detalle |
|---------|--------|
| **Tecnología** | .NET 9 (STS) |
| **Tipo de proyecto** | Class Library + xUnit Tests |
| **Dominio** | Cálculos financieros (interés compuesto, amortización, formateo) |
| **CI/CD** | GitHub Actions con 4 jobs encadenados |
| **Estilo de código** | `.editorconfig` + `dotnet format` |
| **Idioma** | Español (código, documentación, mensajes) |

---

## 🧠 ¿Qué vas a aprender?

```
┌──────────────────────────────────────────────────────────────────┐
│                    WORKSHOP GITHUB INTERMEDIO                     │
│                                                                  │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │  MODULO 1 (60m)  │  │  MODULO 2 (25m)  │  │ MODULO 3(25m)│  │
│  │                  │  │                  │  │              │  │
│  │ GitHub Actions   │  │ Políticas de     │  │ Tags,        │  │
│  │                  │  │ Repositorio      │  │ Releases &   │  │
│  │ • Jobs + needs   │  │                  │  │ Packages     │  │
│  │ • Outputs        │  │ • Branch protect │  │              │  │
│  │ • Artefactos     │  │ • CODEOWNERS     │  │ • Git tags   │  │
│  │ • Matrix builds  │  │ • PR templates   │  │ • SemVer     │  │
│  │ • Step Summary   │  │ • Signed commits │  │ • Releases   │  │
│  │ • Workflow debug │  │ • Merge strategy │  │ • NuGet pack │  │
│  │                  │  │ • Rulesets       │  │ • GH Pkgs    │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
│                                                                  │
│  Proyecto base: FinancialUtils (.NET 9 + xUnit + editorconfig)   │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Pre-requisitos

### Software necesario

```bash
# Verificar instalaciones
dotnet --version   # 9.0 o superior
git --version      # Cualquier versión reciente
gh --version       # GitHub CLI — https://cli.github.com
code --version     # VS Code (recomendado)
```

### Cuenta y permisos

- Una cuenta de GitHub con acceso para crear repositorios públicos
- Git configurado con tu usuario: `git config --global user.email "tu@email.com"`
- GitHub CLI autenticado: `gh auth login`

> 📝 **NOTA:** Si `gh auth login` te pide un token y no tienes uno, ve a GitHub: **Settings > Developer settings > Personal access tokens > Fine-grained tokens**. Necesitas permisos de `repo` y `workflow`.

### Extensiones de VS Code (recomendadas)

1. **GitHub Actions** (GitHub) — Validación de sintaxis YAML para workflows
2. **GitHub Pull Requests** (GitHub) — Gestión de PRs desde VS Code
3. **C# Dev Kit** (Microsoft) — Soporte para .NET

---

## 🚀 Preparación del entorno

```bash
# 1. Hacer fork del repositorio (desde GitHub o con gh)
gh repo fork armandoblanco/workshop-github-intermedio --clone

# 2. Entrar al directorio
cd workshop-github-intermedio

# 3. Verificar que todo compila y pasa
dotnet restore
dotnet build
dotnet test
dotnet format --verify-no-changes
```

> ⚠️ Los **cuatro comandos** deben terminar sin errores. Si alguno falla, revisa la sección de [Troubleshooting](#-troubleshooting) al final.

---

## 🏗️ Arquitectura del proyecto base

```
.
├── 📁 .github/
│   ├── 📁 workflows/
│   │   ├── ci.yml                    # Pipeline: format → build → test → summary
│   │   └── matrix.yml                # Matrix sobre múltiples OS y versiones de .NET
│   ├── 📁 ISSUE_TEMPLATE/
│   │   ├── bug_report.md             # Template para reportar bugs
│   │   └── feature_request.md        # Template para solicitar features
│   ├── CODEOWNERS                    # Reviewers automáticos por área de código
│   └── pull_request_template.md      # Template para PRs
├── 📁 src/
│   └── 📁 FinancialUtils/
│       ├── Calculator.cs             # Interés compuesto, amortización, VPN
│       ├── Formatter.cs              # Moneda, porcentajes, números
│       └── FinancialUtils.csproj
├── 📁 tests/
│   └── 📁 FinancialUtils.Tests/
│       ├── CalculatorTests.cs        # Tests de operaciones financieras
│       ├── FormatterTests.cs         # Tests de formateo
│       └── FinancialUtils.Tests.csproj
├── 📁 exercises/
│   ├── 📁 01-actions-jobs/           # Jobs encadenados, outputs, artefactos, workflow roto
│   ├── 📁 02-matrix-artefactos/      # Matrix strategy, fail-fast, artefactos por combinación
│   ├── 📁 03-branch-protection/      # Branch protection rules, rulesets, estrategias de merge
│   └── 📁 04-tags-releases-packages/ # Tags, releases, NuGet packages, GitHub Packages
├── .editorconfig                     # Reglas de estilo que aplica dotnet format
└── WorkshopGitHub.sln                # Solución .NET
```

### Pipeline CI — Flujo de Jobs

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────────┐
│ Format  │────→│  Build  │────→│  Test   │────→│   Summary   │
│ check   │     │ Release │     │ + Cover │     │ (always())  │
└─────────┘     └─────────┘     └─────────┘     └─────────────┘
     │                               │                  │
     │                               ▼                  ▼
     │                         ┌───────────┐    ┌──────────────┐
     │                         │ coverage- │    │ Tabla con    │
     │                         │ report    │    │ estado de    │
     │                         │ (artifact)│    │ cada job     │
     │                         └───────────┘    └──────────────┘
     │
     └── Si falla → build, test y summary NO corren (needs)
```

---

## 📅 Agenda del Workshop

| Tiempo | Módulo | Contenido | Ejercicios |
|--------|--------|-----------|------------|
| 0:00 — 0:05 | 👋 Introducción | Objetivo, estructura del repo, verificación del entorno | — |
| 0:05 — 0:35 | ⚙️ Módulo 1a | Jobs encadenados, outputs, artefactos | [01-actions-jobs](exercises/01-actions-jobs/README.md) |
| 0:35 — 1:05 | ⚙️ Módulo 1b | Matrix strategy, fail-fast, workflow roto | [02-matrix-artefactos](exercises/02-matrix-artefactos/README.md) |
| 1:05 — 1:10 | ☕ Break | Descanso y Q&A rápido | — |
| 1:10 — 1:35 | 🛡️ Módulo 2 | Branch protection, CODEOWNERS, merge strategies | [03-branch-protection](exercises/03-branch-protection/README.md) |
| 1:35 — 1:55 | 🏷️ Módulo 3 | Tags, Releases y GitHub Packages | [04-tags-releases-packages](exercises/04-tags-releases-packages/README.md) |
| 1:55 — 2:00 | 🎯 Cierre | Recap, preguntas y recursos | — |

> ⏱️ Los tiempos son aproximados. Ajusta según el ritmo del grupo, pero nunca excedas 2 horas.

> 🎓 **Nota para el instructor:** Si la verificación del entorno toma más de 5 minutos, compensa reduciendo el Ejercicio 1.3 (Step Summary). Lo más importante es que los participantes completen los ejercicios de lectura y diagnóstico de los Módulos 1 y 2.

---

## ⚙️ Módulo 1: GitHub Actions (60 min)

El pipeline de este repositorio tiene **cuatro jobs**. Antes de tocar nada, léelo:

```bash
cat .github/workflows/ci.yml
```

### Contenido del módulo

| Concepto | Qué resuelve | Dónde se practica |
|----------|-------------|-------------------|
| `needs` entre jobs | Orden de ejecución y dependencias | Ejercicio 1.1 |
| `outputs` entre jobs | Pasar valores entre runners separados | Ejercicio 1.1, 1.3 |
| `upload-artifact` / `download-artifact` | Pasar archivos entre jobs | Ejercicio 1.2 |
| `$GITHUB_STEP_SUMMARY` | Resumen visual del pipeline | Ejercicio 1.3 |
| Matrix strategy | Probar contra múltiples combinaciones | Ejercicio 2.1, 2.2 |
| `fail-fast` y `exclude` | Control fino del matrix | Ejercicio 2.1 |
| Diagnóstico de workflows | Encontrar errores sin ejecutar | Ejercicio 1.4 |

### Ejercicios

📄 **[Ejercicio 01 — Jobs encadenados, outputs y artefactos](exercises/01-actions-jobs/README.md)**
- 1.1: Leer el pipeline antes de ejecutarlo
- 1.2: Correr el pipeline y revisar artefactos
- 1.3: Agregar conteo de pruebas al Step Summary
- 1.4: Diagnosticar el workflow roto (4 errores)

📄 **[Ejercicio 02 — Matrix builds y artefactos](exercises/02-matrix-artefactos/README.md)**
- 2.1: Analizar el matrix existente
- 2.2: Agregar una dimensión al matrix
- 2.3: Descargar múltiples artefactos con `gh`

---

## 🛡️ Módulo 2: Políticas de repositorio, CODEOWNERS y merge (25 min)

### Contenido del módulo

| Concepto | Qué resuelve | Dónde se practica |
|----------|-------------|-------------------|
| Branch protection rules | Evitar merges sin revisión o con checks fallidos | Ejercicio 3.1 |
| CODEOWNERS | Reviewers automáticos por área de código | Ejercicio 3.2 |
| PR e Issue templates | Estandarizar la comunicación del equipo | Ejercicio 3.3 |
| Commits firmados (SSH) | Autenticidad del autor | Ejercicio 3.4 |
| Merge strategies | Historial limpio vs. granular | Ejercicio 3.5 |
| Rulesets vs. rules clásicas | Políticas escalables | Referencia |

### Ejercicios

📄 **[Ejercicio 03 — Branch protection y políticas](exercises/03-branch-protection/README.md)**
- 3.1: Configurar branch protection en `main`
- 3.2: CODEOWNERS para reviewers automáticos
- 3.3: Templates de PR e issues
- 3.4: Commits firmados con SSH
- 3.5: Comparar squash vs. merge vs. rebase

---

## 🏷️ Módulo 3: Tags, Releases y GitHub Packages (25 min)

### Contenido del módulo

| Concepto | Qué resuelve | Dónde se practica |
|----------|-------------|-------------------|
| Tags ligeros vs. anotados | Marcar puntos específicos del historial | Ejercicio 4.1 |
| Versionamiento semántico (SemVer) | Comunicar el tipo de cambio en cada versión | Ejercicio 4.1 |
| GitHub Releases | Distribuir el código con notas y binarios adjuntos | Ejercicio 4.2 |
| Workflow de publicación | Automatizar `dotnet pack` + push a GitHub Packages | Ejercicio 4.3 |
| Consumir paquetes | Usar paquetes internos desde otro proyecto | Ejercicio 4.4 |

### Ejercicios

📄 **[Ejercicio 04 — Tags, Releases y Packages](exercises/04-tags-releases-packages/README.md)**
- 4.1: Tags con Git y versionamiento semántico
- 4.2: Crear un GitHub Release con `gh`
- 4.3: Publicar a GitHub Packages con un workflow
- 4.4: Consumir un paquete desde GitHub Packages

---

## ✅ Checklist final

Al terminar el workshop, deberías poder:

### GitHub Actions
- [ ] Leer un workflow YAML y entender el flujo de jobs
- [ ] Explicar para qué sirve `needs`, `outputs` y `upload-artifact`
- [ ] Diseñar un matrix strategy con `exclude` y `fail-fast`
- [ ] Diagnosticar errores comunes en workflows sin ejecutarlos
- [ ] Usar `$GITHUB_STEP_SUMMARY` para generar reportes

### Políticas de repositorio
- [ ] Configurar branch protection rules en `main`
- [ ] Escribir un archivo CODEOWNERS con owners por área
- [ ] Explicar la diferencia entre squash, merge commit y rebase
- [ ] Configurar commits firmados con SSH
- [ ] Crear templates de PR e issues

### Tags, Releases y Packages
- [ ] Crear tags anotados con versionamiento semántico
- [ ] Crear un GitHub Release con notas automáticas
- [ ] Configurar un workflow que publique a GitHub Packages
- [ ] Entender cuándo usar GitHub Packages vs. NuGet.org
- [ ] Consumir un paquete publicado desde otro proyecto

---

## 🔧 Troubleshooting

| Problema | Solución |
|----------|----------|
| `dotnet restore` falla con error de conexión | Verifica acceso a nuget.org. En entornos corporativos con proxy, puede requerir `NuGet.Config` |
| `dotnet format --verify-no-changes` falla | Ejecuta `dotnet format` sin `--verify-no-changes` para ver qué cambia. Verifica que tu editor usa el `.editorconfig` del proyecto |
| `dotnet test` no encuentra los tests | Ejecuta desde la raíz: `ls WorkshopGitHub.sln && dotnet restore && dotnet build && dotnet test` |
| El workflow no se dispara al hacer push | El CI solo corre en `main`, `feature/**` y `fix/**`. Una rama llamada `mi-rama` no lo dispara |
| `gh auth login` pide token | Ve a GitHub: **Settings > Developer settings > Personal access tokens > Fine-grained tokens**. Necesitas permisos `repo` y `workflow` |
| `gh` no está instalado | Instala desde https://cli.github.com — en macOS: `brew install gh` |
| Error "No hosted runner found" | Verifica que el nombre del runner sea exacto: `ubuntu-latest` (no `ubuntu-latests`) |
| El CODEOWNERS no solicita reviews | Verifica que los usuarios tengan acceso de escritura al repositorio y que "Require review from Code Owners" esté activado |

---

## 📚 Recursos adicionales

### Documentación oficial

- [GitHub Actions](https://docs.github.com/en/actions) — Referencia completa de workflows
- [GitHub CLI](https://cli.github.com/manual) — Manual de `gh`
- [GitHub Skills](https://skills.github.com) — Laboratorios guiados oficiales
- [GitHub Rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets) — Políticas modernas de repositorio

### .NET y herramientas

- [actions/setup-dotnet](https://github.com/actions/setup-dotnet) — Acción oficial para .NET en CI
- [XPlat Code Coverage](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-code-coverage) — Cobertura con `dotnet test`
- [Versiones soportadas de .NET](https://dotnet.microsoft.com/platform/support/policy/dotnet-core) — Ciclo de vida LTS y STS
- [EditorConfig](https://editorconfig.org/) — Reglas de estilo portables entre editores

### Para profundizar después del workshop

- **GitHub Actions avanzado:** Composite actions, reusable workflows, environments con approval gates
- **Seguridad en CI:** Dependabot, CodeQL, secret scanning, OIDC para deploys sin secrets
- **Métricas de equipo:** DORA metrics, deployment frequency, lead time con GitHub
- **GitHub Copilot + CLI:** `gh copilot suggest` para generar comandos con IA

---

## 🙋 Preguntas frecuentes

### ¿Por qué .NET 9 y no otra versión?

.NET 9 es la versión STS más reciente (noviembre 2024). La usamos como target principal para estar al día con las últimas APIs, mientras el matrix de CI también incluye .NET 8 (LTS, soporte hasta 2026) para verificar compatibilidad.

### ¿Necesito saber C# para este workshop?

No. El código C# es deliberadamente simple (operaciones matemáticas y formateo). El foco está en GitHub Actions, branch protection y GitHub CLI. Solo necesitas poder ejecutar `dotnet build` y `dotnet test`.

### ¿Puedo usar este material para dar un workshop en mi empresa?

Sí. El repositorio es público y el contenido está diseñado para ser reutilizable. Recomendamos hacer un fork, reemplazar `@TU_USUARIO` en CODEOWNERS con los usuarios reales, y adaptar los tiempos según tu audiencia.

### ¿Por qué GitHub CLI y no solo la web?

`gh` se usa extensivamente en este workshop para crear tags, releases y monitorear workflows. Su valor está en: (1) mantener el foco sin romper el contexto de la terminal, (2) scripting para automatizar flujos como crear releases y subir artefactos, y (3) es la forma más rápida de interactuar con GitHub Packages.

### ¿El workflow roto es intencional?

Sí. `exercises/01-actions-jobs/workflow-roto.yml` tiene 4 errores intencionales que replican errores reales que se encuentran en proyectos profesionales. El objetivo es practicar lectura y diagnóstico de YAML sin necesidad de ejecutar.

### ¿Qué pasa si no termino todos los ejercicios?

No te preocupes. Los módulos son independientes:
- Si no terminas el Módulo 1, puedes pasar al Módulo 2 sin problema
- El Módulo 3 (Tags/Releases/Packages) es el más flexible y puede adaptarse al tiempo disponible
- El valor está en entender los conceptos, no en completar todos los ejercicios

---

## 🆘 ¿Te quedaste atrás?

| Situación | Qué hacer |
|-----------|-----------|
| No terminé el Ejercicio 1 | Los ejercicios son independientes — pasa al siguiente |
| El workflow no se dispara | Verifica el nombre de la rama: debe ser `feature/**` o `fix/**` |
| No entiendo el YAML | Pide al instructor una explicación paso a paso del `ci.yml` |
| Mi `gh` no está autenticado | Ejecuta `gh auth login` y sigue las instrucciones |
| Resultados diferentes a mi compañero | Eso es normal — cada fork tiene su propio contexto de Actions |

> 🎓 **Para el instructor:** Se recomienda tener una rama `solucion` con los workflows modificados según los ejercicios. Esto permite que participantes que se queden atrás puedan ver las respuestas y seguir avanzando.

---

## 👥 Créditos

**Workshop desarrollado por:** [Armando Blanco](https://github.com/armandoblanco)

**Tecnologías:** GitHub Actions, GitHub CLI, .NET 9, xUnit, EditorConfig, CODEOWNERS, Branch Protection Rules

**Duración:** 2 horas

**Nivel:** Intermedio (asume conocimiento básico de Git y GitHub)

> 🎉 **¡Gracias por participar!** Ahora tienes las herramientas para llevar tus repositorios de "funciona" a "escala con el equipo".
