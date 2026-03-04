---
name: Commit Spelunker
description: Herramienta especializada de análisis exhaustivo para explorar el historial de repositorios git, descubriendo patrones de colaboradores, tendencias de evolución de código y hallazgos ocultos mediante consultas y visualizaciones avanzadas de git.
version: 1.2.0
author: SMOUJBOT
tags:
  - git
  - history
  - research
  - patterns
  - analytics
  - forensics
dependencies:
  - git >= 2.30
  - bash >= 4.0
  - coreutils
  - grep
  - sed
  - awk
  - dateutils (optional)
  - gnuplot (optional, for graphs)
  - gitstats (optional, for reports)
env_vars:
  - COMMIT_SPELUNKER_MAX_COMMITS: Limita el análisis de historial (por defecto: 10000)
  - COMMIT_SPELUNKER_DATE_FORMAT: Formato de fecha de salida (por defecto: %Y-%m-%d)
  - COMMIT_SPELUNKER_LOGGING: Habilita logging de depuración (0/1, por defecto: 0)
---

# Commit Spelunker

Una habilidad especializada para exploración exhaustiva del historial de repositorios git. Extrae patrones significativos de datos de commits, analiza comportamiento de colaboradores, rastrea evolución de archivos y genera hallazgos accionables sobre salud del codebase y prácticas de desarrollo.

## Propósito

Casos de uso reales:

- **Investigación de onboarding**: Comprender la evolución del codebase antes de realizar cambios arquitectónicos
- **Caza de bugs**: Identificar cuándo y dónde se introdujeron patrones de código específicos
- **Análisis de equipo**: Medir contribuciones entre equipos/individuos, identificar silos de conocimiento
- **Seguimiento de deuda técnica**: Encontrar commits de "arreglo rápido" (cambios de una línea, mensajes "hotfix")
- **Forensia de releases**: Reconstruir qué cambió entre releases específicas
- **Detección de decaimiento arquitectónico**: Encontrar archivos con mayor tasa de cambio (modificados frecuentemente)
- **Mapeo de colaboradores**: Identificar quién posee qué partes del codebase mediante frecuencia de commits
- **Predicción de conflictos de merge**: Identificar archivos con altas tasas de modificación concurrente
- **Auditoría de compliance**: Rastrear cuándo se agregaron licencias específicas, cabeceras de copyright o patrones de seguridad
- **Detección de código muerto**: Encontrar archivos/features que no se han tocado en años

## Alcance

Operaciones soportadas:

```bash
# Búsqueda de patrones en mensajes de commit
commits-by-message --pattern "bugfix|hotfix|quick" --since "6 months ago"
commits-by-message --pattern "refactor" --author "john" --exclude "docs"

# Análisis de colaboradores
contributor-stats --top 10 --metric commits
contributor-stats --active-days --per-file
contributor-heatmap --since "1 year ago"

# Seguimiento de evolución de archivos
file-history <path/to/file> --detailed
file-churn --top 20 --min-commits 5
file-ownership <path> --show-blame-summary

# Análisis basado en tiempo
commit-rhythm --interval weekly --heatmap
commit-punch-card --last 6 months
dead-code-finder --stale-years 2

# Análisis de relaciones de código
coupling-finder <path> --co-modified-with
feature-birth-tracking --pattern "feature:.*login"

# Arqueología de releases
diff-between-tags v1.2.0 v1.3.0 --stats-only
changes-under-tag <tag> --group-by-author

# Análisis de merges
merge-forensics --complex-merges --show-conflicts
branch-lifespan-analysis --merged-only
```

## Proceso de Trabajo Detallado

### Fase 1: Evaluación del Repositorio
```bash
# Verificación rápida de salud
git rev-parse --is-inside-work-tree
git remote -v
git branch -a | wc -l
git log --oneline | wc -l
```

### Fase 2: Panorama de Colaboradores
```bash
# Principales colaboradores por count de commits
git shortlog -sne --all | head -20

# Recencia de contribuciones (últimos 90 días)
git log --since="90 days ago" --pretty="%an" | sort | uniq -c | sort -rn | head -15

# Frecuencia de commits por día de la semana/hora
git log --pretty="%ad" --date=local | awk '{print $4}' | cut -d: -f1 | sort | uniq -c
```

### Fase 3: Análisis Profundo de Archivos
```bash
# Archivos más cambiados (alto churn)
git log --pretty=format: --name-only | sort | uniq -c | sort -rg | head -20

# Archivos con más autores distintos
git log --pretty="%an" --name-only | awk 'NF==1{file=$0;next}{print file":"$0}' | \
  sort | uniq | awk -F: '{count[$1]++} END {for (f in count) print count[f], f}' | sort -rn | head -15

# Commits más grandes (por count de archivos)
git log --oneline --numstat | awk 'NF==3 {add+=$1+$2; files++} END {print files, add}'
```

### Fase 4: Detección de Patrones
```bash
# Encontrar commits "work in progress" o incompletos
git log --grep="WIP\|TODO\|FIXME\|XXX" --oneline

# Identificar arreglos rápidos (cambios de una línea)
git log --oneline --numstat | awk 'NF==3 && ($1+$2)==1 {print $0}'

# Rastrear ciclo de vida de desarrollo de features
git log --grep="^feat(.*)" --oneline | wc -l
```

### Fase 5: Análisis Temporal
```bash
# Patrón mensual de commits (para planificación de capacidad)
git log --since="2 years ago" --pretty="%s%ad" --date=format:%Y-%m | sort | uniq -c

# Tiempo entre commits (velocidad del desarrollador)
git log --pretty="%at" | awk 'NR>1 {print $1-prev} {prev=$1}' | \
  awk '{sum+=$1; count++} END {print "Avg seconds between commits:", sum/count}'
```

### Fase 6: Mapeo de Relaciones
```bash
# Archivos cambiados frecuentemente juntos (acoplamiento)
git log --pretty=format: --name-only | awk 'NF>0 {file=$0; getline; while (getline && NF>0) print file","$0}' | \
  tr ',' '\n' | sort | uniq -c | sort -rn | head -20

# Patrones de merge de ramas
git log --merges --oneline --pretty="%s" | cut -d'(' -f2 | cut -d')' -f1 | sort | uniq -c
```

### Fase 7: Reporte y Exportación
```bash
# Exportar a CSV para análisis en spreadsheet
git log --pretty="%H,%an,%ae,%ad,%s" --date=iso > commits_export.csv

# Generar visualización (si gitstats disponible)
gitstats /path/to/repo /output/directory
```

## Reglas de Oro

1. **Conciencia de rendimiento**: Siempre usar `--since`, `--until`, `--max-count` en repos grandes. Límite por defecto: `COMMIT_SPELUNKER_MAX_COMMITS=10000`
2. **Respetar .gitattributes**: Honrar configuración de detección de renombrados (`-M` activa/desactiva)
3. **Preservar contexto**: Nunca incluir diffs de archivos completos en logs; usar `--stat` en lugar de diff completo
4. **Atribución de autor**: Usar email canónico (`git shortlog -sne` muestra mapeo autoritativo)
5. **Zonas horarias**: Normalizar a UTC para análisis cross-región (`--date=iso-strict`)
6. **Archivos binarios**: Excluir del análisis de churn (aparecen como "0" líneas cambiadas)
7. **Commits de merge**: Decidir de antemano si incluirlos (`--no-merges` vs `--merges`)
8. **Seguridad**: Nunca mostrar hashes de commit completos en reportes públicos; truncar a 8 caracteres
9. **Reproducibilidad**: Todos los scripts de análisis deben loggear comando git exacto usado
10. **No destructivo**: Todas las operaciones son solo lectura; nunca hacer checkout o reset durante la exploración

## Ejemplos

### Ejemplo 1: Encontrar archivos más controversiales (más autores)
```bash
# Entrada
commit-spelunker file-ownership --path src/ --min-authors 3

# Salida
   12 src/utils/helpers.js
    8 src/components/Button.tsx
    7 src/services/api.ts
    5 src/hooks/useAuth.ts
```

### Ejemplo 2: Detectar commits "arreglo rápido" (probable deuda técnica)
```bash
# Entrada
commit-spelunker pattern-search --pattern "quick|hotfix|temporary" --exclude "docs|test" --last 30

# Salida
a1b2c3d4 (2025-03-01) alice: Quick fix for login crash
e5f6g7h8 (2025-02-28) bob: Hotfix: race condition in checkout
i9j0k1l2 (2025-02-27) charlie: Temporary workaround for Safari
```

### Ejemplo 3: Punch card de colaborador (QUIÉN programa CUÁNDO)
```bash
# Entrada
commit-spelunker punch-card --last 12 weeks --output matrix.tsv

# Salida (TSV):
Mon Tue Wed Thu Fri Sat Sun
  15  12  18  14  11   3   1  # Semana 1
  14  16  13  17  12   2   0  # Semana 2
...
```

### Ejemplo 4: Seguimiento de nacimiento de feature
```bash
# Entrada
commit-spelunker feature-tracking --regex "^feat:.*payment" --show-evolution

# Salida
Feature: payment-processing
  Born:  2024-08-15 (commit 7a8b9c0) - "feat: add Stripe integration"
  Matured: 2024-10-22 (commit d1e2f3a) - "feat(payment): webhook handling"
  Stabilized: 2025-01-05 (commit 4g5h6i7) - "fix: idempotency keys"
  Current owners: alice (68%), bob (22%), charlie (10%)
```

### Ejemplo 5: Mapa de calor de churn para priorización de refactor
```bash
# Entrada
commit-spelunker churn-heatmap --top 15 --format json > churn.json

# Salida (extracto JSON):
{
  "files": [
    {"path": "src/store/cart.ts", "commits": 142, "authors": 8, "lines_changed": 3847},
    {"path": "src/components/ProductGrid.js", "commits": 128, "authors": 6, "lines_changed": 2912}
  ]
}
```

## Comandos de Rollback

Commit Spelunker es de solo lectura por diseño. Las acciones de rollback solo aplican a archivos temporales:

```bash
# Eliminar salidas de análisis temporales
rm -f /tmp/commit-spelunker-*.{csv,tsv,json,txt}

# Limpiar entradas de reflog de git creadas durante la exploración (ninguna por defecto)
# Nota: La habilidad nunca crea entradas de reflog

# Restablecer cambios de permisos de archivos (si se ejecuta con permisos elevados)
chmod -R u+rwX,go+rX /tmp/commit-spelunker-*/

# Si se usa gitstats y se quitan reportes generados
rm -rf /var/tmp/gitstats-output/

# Limpieza de entorno
unset COMMIT_SPELUNKER_*
```

## Pasos de Verificación

Después de ejecutar cualquier operación de Commit Spelunker:

```bash
# 1. Verificar código de salida
echo $?
# Debería ser 0 para éxito

# 2. Verificar que archivo de salida existe (si se usó --output)
test -f analysis.csv && echo "Output created" || echo "No output"

# 3. Validar formato CSV (si aplica)
head -1 analysis.csv | grep -q "," && echo "CSV valid" || echo "Invalid CSV"

# 4. Asegurar que no hay secrets filtrados en salida
grep -iE "(password|token|secret|key)" analysis.* && echo "WARNING: Secrets detected!" || echo "Clean"

# 5. Confirmar integridad del repo git preservada
git status --short | grep -q '^M\|^D\|^A' && echo "Repo modified!" || echo "Repo clean"
```

## Solución de Problemas

**Síntoma**: "fatal: ambiguous argument 'HEAD'"
- **Causa**: No estar en un repositorio git
- **Fix**: `cd /path/to/repo` antes de ejecutar la habilidad

**Síntoma**: "Argument list too long" en repos grandes
- **Causa**: Exceder límite de argumentos de shell
- **Fix**: Establecer `COMMIT_SPELUNKER_MAX_COMMITS=5000` o usar flag `--batch-size`

**Síntoma**: Commits de merge sesgan estadísticas de autor
- **Causa**: Commits de merge contados como commits regulares
- **Fix**: Añadir flag `--no-merges` o establecer `COMMIT_SPELUNKER_EXCLUDE_MERGES=1`

**Síntoma**: Detección de renombrados no funciona
- **Causa**: Detección de renombrados de git deshabilitada globalmente
- **Fix**: Usar flag `--detect-renames` manualmente o revisar `.gitattributes`

**Síntoma**: Rendimiento lento en monorepo
- **Causa**: Analizar historial completo (500k+ commits)
- **Fix**: Usar `--since "1 year ago"` o `--max-count 10000`

**Síntoma**: Archivos binarios apareciendo en análisis de churn
- **Causa**: Archivos binarios contados como "1 línea cambiada"
- **Fix**: Pasar mediante `--exclude "*.png,*.jpg,*.pdf"` o usar script de filtro

**Síntoma**: Fechas en zona horaria incorrecta
- **Causa**: Config local de git vs requisito UTC
- **Fix**: Establecer `export GIT_AUTHOR_DATE=UTC` o usar `--date=iso-strict`
```