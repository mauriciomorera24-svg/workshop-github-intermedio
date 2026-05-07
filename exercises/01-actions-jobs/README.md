# ⚙️ Módulo 1: GitHub Actions — Jobs encadenados, outputs y artefactos

![Duración](https://img.shields.io/badge/Duración-60%20min-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Intermedio-orange)

## 🎯 Objetivos

- ✅ Entender dependencias entre jobs con `needs`
- ✅ Pasar valores entre jobs con `outputs`
- ✅ Pasar archivos entre jobs con `upload-artifact` / `download-artifact`
- ✅ Generar resúmenes visuales con `$GITHUB_STEP_SUMMARY`
- ✅ Diagnosticar errores comunes en workflows sin ejecutarlos

---

## Contexto

La diferencia entre un pipeline que funciona y uno que escala está en cómo estructuras los jobs: qué depende de qué, qué información pasa entre ellos y qué artefactos produces. Este módulo cubre esas tres cosas con el pipeline real de este repositorio, que corre `dotnet format`, `dotnet build` y `dotnet test` en jobs separados con dependencias explícitas.

---

## Ejercicio 1.1 — Leer el pipeline antes de ejecutarlo (15 min)

Abre `.github/workflows/ci.yml` y responde:

1. ¿Qué job se ejecuta primero? ¿Por qué no hay un job de restore independiente?
2. ¿Qué impediría que el job `test` corra si `build` falla?
3. ¿Qué valor está pasando el job `format` al job `build` mediante `outputs`? ¿Dónde lo consume `build`?
4. ¿En qué casos corre el job `summary`? Presta atención a `if: always()`.
5. ¿Qué diferencia hay entre el artefacto `coverage-report` y `test-results`? ¿Por qué uno usa `if: always()` y el otro no?

**Sin ejecutar nada todavía.** El objetivo es leer YAML como documentación.

### Conceptos clave

**`needs`** define dependencias explícitas. Sin él, los jobs corren en paralelo y no hay garantía de orden.

```yaml
test:
  needs: build   # test espera a que build termine con éxito
```

**`outputs`** permite exponer valores entre jobs. Los jobs corren en runners separados, así que no comparten variables de entorno ni sistema de archivos.

```yaml
jobs:
  format:
    outputs:
      dotnet-version: ${{ steps.setup-dotnet.outputs.dotnet-version }}
    steps:
      - id: setup-dotnet
        uses: actions/setup-dotnet@v4
        ...

  build:
    needs: format
    steps:
      - run: echo "Versión de .NET: ${{ needs.format.outputs.dotnet-version }}"
```

**`upload-artifact` / `download-artifact`** pasan archivos entre jobs. Cada job corre en un runner limpio sin acceso al sistema de archivos de los otros jobs.

---

## Ejercicio 1.2 — Correr el pipeline y revisar los artefactos (15 min)

1. Clona el repositorio e instala dependencias:

```bash
git clone https://github.com/TU_USUARIO/workshop-github-intermedio.git
cd workshop-github-intermedio
dotnet restore
dotnet build
dotnet test
```

2. Crea una rama y dispara el workflow:

```bash
git checkout -b feature/primer-ejercicio
echo "// ejercicio" >> src/FinancialUtils/Calculator.cs
# Esto romperá dotnet format --verify-no-changes porque agrega contenido fuera de estilo
```

En lugar de agregar un comentario directamente, haz un cambio válido:

```bash
git checkout -b feature/primer-ejercicio
# Modifica el XML del csproj: agrega un Description al PropertyGroup
git add .
git commit -m "feat: primer ejercicio del workshop"
git push origin feature/primer-ejercicio
```

3. Ve a la pestaña **Actions** en GitHub y observa:
   - El orden de ejecución de los jobs
   - Qué pasa si `format` falla: ¿llegan a correr `build` y `test`?
   - El contenido del **Step Summary** generado por el job `summary`

4. Descarga el artefacto `coverage-report`. Dentro encontrarás un XML de cobertura en formato Cobertura (el que genera `XPlat Code Coverage`). En un pipeline real se convierte a HTML con herramientas como ReportGenerator.

**Punto de discusión:** ¿Por qué el job `test` vuelve a hacer `dotnet restore` si `build` ya lo hizo? ¿Qué pasaría si el job `build` subiera los binarios como artefacto y `test` los descargara?

---

## Ejercicio 1.3 — Agregar conteo de pruebas al Step Summary (15 min)

Modifica `.github/workflows/ci.yml` para que el job `test` exponga como output la cantidad de pruebas ejecutadas, y que el job `summary` lo muestre en el Step Summary.

Puedes extraer el número del archivo `.trx` que genera `dotnet test`:

```bash
# El archivo .trx es XML. Con grep simple:
TOTAL=$(grep -oP 'total="\K[^"]+' coverage/**/*.trx | head -1)
echo "total-tests=$TOTAL" >> $GITHUB_OUTPUT
```

El job `summary` debería mostrar algo como:

```
| Pruebas ejecutadas | 35 |
```

---

## Ejercicio 1.4 — Diagnosticar el workflow roto (15 min)

Abre `exercises/01-actions-jobs/workflow-roto.yml`. Tiene **4 errores intencionales**. Encuéntralos sin ejecutar el archivo.

> 💡 **Tip:** Los errores son de configuración, no de lógica de negocio. Busca typos, nombres de referencia incorrectos y rutas que no existen.

Anota cada error y su corrección antes de revisar `exercises/01-actions-jobs/solucion.md`.

Si necesitas más de 10 minutos para encontrar los 4, revisa la solución y entiende por qué cada uno falla. Estos patrones se repiten en proyectos reales.

### Pistas (solo si te atoras)

<details>
<summary>🔍 Pista 1</summary>
Revisa el trigger <code>on.push</code> — ¿la clave está en singular o plural?
</details>

<details>
<summary>🔍 Pista 2</summary>
Verifica el nombre del runner — ¿está escrito exactamente como lo espera GitHub?
</details>

<details>
<summary>🔍 Pista 3</summary>
El job <code>test</code> declara una dependencia con <code>needs</code> — ¿el nombre del job referenciado coincide con el nombre real?
</details>

<details>
<summary>🔍 Pista 4</summary>
¿La ruta del artefacto de cobertura coincide con el directorio donde <code>dotnet test</code> genera los resultados?
</details>

---

## 🛠️ Troubleshooting del Módulo

| Problema | Solución |
|----------|----------|
| El workflow no se dispara | Verifica que la rama siga el patrón `feature/**` o `fix/**` |
| El job `build` no corre | Revisa que `format` haya pasado — `needs` bloquea si el job previo falla |
| El artefacto se sube vacío | Verifica la ruta en `path:` y que `dotnet test` genere los archivos esperados |
| El Step Summary no se ve | Busca la pestaña "Summary" en el run de Actions, no en los logs del job |
| `grep` no encuentra el archivo `.trx` | Verifica la ruta `coverage/**/*.trx` — puede variar según la versión de .NET |

---

## ✅ Verificación

Al terminar este módulo, deberías poder responder:

1. ¿Qué diferencia hay entre `outputs` y `upload-artifact`?
2. ¿Por qué el job `summary` usa `if: always()`?
3. ¿Cuántos de los 4 errores del workflow roto encontraste antes de ver la solución?

> 📝 **Para el instructor:** El Ejercicio 1.4 (workflow roto) es el de mayor impacto práctico. Si el tiempo es limitado, prioriza este sobre el 1.3.
