# 🖥️ Módulo 3: GitHub CLI

![Duración](https://img.shields.io/badge/Duración-25%20min-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Intermedio-orange)

## 🎯 Objetivos

- ✅ Crear y gestionar PRs sin abrir el navegador
- ✅ Monitorear el pipeline de CI en tiempo real desde terminal
- ✅ Hacer checkout de PRs de otros colaboradores con un solo comando
- ✅ Aprobar, solicitar cambios y comentar desde terminal
- ✅ Descargar artefactos de Actions y crear issues con scripting

---

## Contexto

`gh` no es solo una alternativa al navegador. Su valor real está en dos cosas: mantener el foco (no romper el contexto de la terminal mientras trabajas en código) y scripting (automatizar flujos que en la UI requieren múltiples clics). Este módulo cubre los comandos con mayor impacto en el día a día de un equipo .NET.

---

## Prerequisitos

```bash
# Verificar instalación
gh --version

# Autenticarse (solo la primera vez)
gh auth login
```

Si no tienes `gh` instalado: https://cli.github.com

---

## Ejercicio 4.1 — Crear y gestionar PRs desde terminal (10 min)

```bash
git checkout -b feature/cli-ejercicio
# Agrega algo mínimo a Calculator.cs
git add .
git commit -m "feat: ejercicio de GitHub CLI"
git push origin feature/cli-ejercicio

# Crear el PR sin abrir el navegador
gh pr create \
  --title "feat: ejercicio de GitHub CLI" \
  --body "PR creado desde terminal como parte del workshop." \
  --base main \
  --head feature/cli-ejercicio

# Ver el estado del PR
gh pr status

# Ver el PR con checks en terminal
gh pr view

# Abrir en navegador si necesitas ver algo visual
gh pr view --web
```

**Con template:** Si tu repo tiene `.github/pull_request_template.md`, `gh pr create` lo usa automáticamente si omites `--body`. Prueba crear uno y vuelve a ejecutar `gh pr create` sin `--body`.

---

## Ejercicio 4.2 — Monitorear el pipeline sin salir de la terminal (5 min)

```bash
# Ver los checks del PR de la rama actual
gh pr checks

# Esperar en tiempo real a que terminen (útil mientras trabajas en otra cosa)
gh pr checks --watch

# Ver los últimos runs de Actions
gh run list --limit 5

# Ver el log completo de un run
gh run view               # interactivo — selecciona con flechas
gh run view [RUN_ID] --log

# Filtrar por workflow
gh run list --workflow=ci.yml --limit 5
```

`gh pr checks --watch` hace polling y actualiza el estado en la terminal. Evita refrescar el navegador cada 30 segundos en pipelines de 2-5 minutos como el de este workshop.

---

## Ejercicio 4.3 — Checkout de un PR de otro colaborador (5 min)

Este comando ahorra más tiempo que cualquier otro en revisiones de código:

```bash
# Checkout por número de PR
gh pr checkout 42

# Checkout por URL
gh pr checkout https://github.com/OWNER/REPO/pull/42

# Listar PRs y hacer checkout interactivo
gh pr list
gh pr checkout   # sin argumentos: muestra selector

# Después de revisar, volver a tu rama
git checkout -
```

`gh pr checkout` hace el fetch de la rama remota y el checkout local en un solo comando, sin tener que configurar el remote ni conocer el nombre de la rama. Equivale a:

```bash
git fetch origin pull/42/head:pr-42
git checkout pr-42
```

---

## Ejercicio 4.4 — Flujo completo de revisión desde terminal (5 min)

```bash
# Aprobar un PR
gh pr review --approve

# Solicitar cambios con comentario
gh pr review --request-changes --body "Falta validar que annualRate no sea mayor a 1 en LoanPayment."

# Comentar sin bloquear
gh pr review --comment --body "¿Consideraste usar checked arithmetic aquí para detectar overflow?"

# Mergear con squash y eliminar la rama remota
gh pr merge --squash --delete-branch

# Descargar artefactos del último run
RUN_ID=$(gh run list --workflow=ci.yml --limit=1 --json databaseId --jq '.[0].databaseId')
gh run download $RUN_ID --name coverage-report

# Crear un issue desde terminal
gh issue create \
  --title "Bug: LoanPayment con tasa 100% genera resultado inesperado" \
  --body "Pasos para reproducir: Calculator.LoanPayment(10000, 1.0m, 12)" \
  --label bug

# Referenciar el issue en un commit para cerrarlo automáticamente al mergear
git commit -m "fix: agregar validación de tasa máxima en LoanPayment

Closes #15"
```

---

## Referencia rápida

| Tarea | Comando |
|-------|---------|
| Crear PR | `gh pr create` |
| Ver estado del PR | `gh pr status` |
| Ver checks | `gh pr checks` |
| Monitorear en vivo | `gh pr checks --watch` |
| Checkout de PR | `gh pr checkout [número]` |
| Aprobar PR | `gh pr review --approve` |
| Solicitar cambios | `gh pr review --request-changes` |
| Mergear PR | `gh pr merge` |
| Ver runs | `gh run list` |
| Ver log de run | `gh run view [id] --log` |
| Descargar artefacto | `gh run download [id]` |
| Crear issue | `gh issue create` |

Documentación completa: `gh help` o https://cli.github.com/manual

---

## 🛠️ Troubleshooting del Módulo

| Problema | Solución |
|----------|----------|
| `gh: command not found` | Instala con `brew install gh` (macOS) o `winget install GitHub.cli` (Windows) |
| `gh auth login` falla | Verifica conexión a internet y que el token tenga scopes `repo`, `read:org` |
| `gh pr create` dice "no commits between..." | Verifica que tu rama tenga commits que `main` no tiene (`git log main..HEAD`) |
| `gh pr checks --watch` no muestra nada | El PR debe tener un workflow que se dispare. Verifica los triggers del CI |
| `gh run download` falla | Verifica que el run generó artefactos (no todos los workflows suben artefactos) |
| `gh pr checkout` da error de merge | Haz `git stash` antes del checkout si tienes cambios sin commit |

---

## ✅ Verificación

Al terminar este módulo, deberías poder responder:

1. ¿En qué escenarios `gh` es más eficiente que la interfaz web?
2. ¿Qué diferencia hay entre `gh pr merge --squash` y `gh pr merge --rebase`?
3. ¿Cómo combinarías `gh` con scripts de bash para automatizar tu flujo de trabajo?
4. ¿Cómo cerrarías un issue automáticamente al mergear un PR?

> 📝 **Para el instructor:** El Ejercicio 4.1 (crear PR) y 4.2 (monitorear pipeline) son los que más impacto tienen. Si el tiempo es limitado, el 4.4 puede ser una demo en vivo del instructor.
