# Propuesta de Trabajo — Estrategia de Ramas y Flujo Git

> **Resumen**: Dos ramas independientes sobre un fork. `gemelo-digital` para features propias del DT. `contribution` para PRs al upstream original.

## Estructura de Ramas

```
upstream (boinor/boinor)
    │
    │  fetch
    ▼
origin (JuanJo-Saavedra/boinor)
    │
    ├── main              ← base estable, sincronizada con upstream
    ├── gemelo-digital    ← features del DT (NO van a upstream)
    └── contribution      ← fixes/mejoras (SÍ van como PR a upstream)
```

| Rama | Propósito | Destino del código |
|------|-----------|-------------------|
| `main` | Base estable | Se sincroniza con upstream |
| `gemelo-digital` | Telemetría, subsistemas, eventos, anomalías | Se queda en tu fork |
| `contribution` | Bugs, mejoras generales de boinor | PR al repo original |

---

## Configuración Inicial

### Remotes

```bash
# Tu fork (ya configurado como origin)
origin   → https://github.com/JuanJo-Saavedra/boinor.git

# Repo original (configurado como upstream)
upstream → https://github.com/boinor/boinor.git
```

### Verificar configuración

```bash
git remote -v
```

---

## Flujo de Actualización (upstream → gemelo-digital)

Cuando el repo original tenga cambios nuevos:

```bash
# 1. Traer los últimos cambios del repo original
git fetch upstream
#    → Descarga commits de upstream sin modificar tus ramas locales

# 2. Cambiar a main
git checkout main
#    → Te posiciona en la rama base

# 3. Fusionar los cambios de upstream en main
git merge upstream/main
#    → Integra las novedades del repo original en tu main local

# 4. Subir main actualizado a tu fork en GitHub
git push origin main
#    → Sincroniza tu fork remoto con los cambios de upstream

# 5. Cambiar a gemelo-digital
git checkout gemelo-digital
#    → Te posiciona en la rama del DT

# 6. Fusionar main (con las novedades de upstream) en gemelo-digital
git merge main
#    → Trae las mejoras de boinor a tu rama de DT

# 7. Subir gemelo-digital actualizado a tu fork
git push origin gemelo-digital
#    → Sincroniza tu rama de DT en el fork remoto
```

### Diagrama del flujo

```
upstream/main ──fetch──→ FETCH_HEAD
                            │
                         merge
                            ▼
                     origin/main (local)
                            │
                    ┌───────┴───────┐
                    │               │
                 push            merge
                    │               │
                    ▼               ▼
            origin/main      gemelo-digital
            (remoto)              │
                               push
                                  │
                                  ▼
                        origin/gemelo-digital
                            (remoto)
```

---

## Flujo de Contribución (contribution → upstream)

Cuando quieras contribuir al repo original:

```bash
# 1. Asegurarte de que main está actualizado
git checkout main
git fetch upstream
git merge upstream/main

# 2. Crear rama de contribution desde main actualizado
git checkout contribution
git merge main
#    → contribution parte de la última versión de upstream

# 3. Hacer tus cambios y commitear
git add .
git commit -m "fix: descripción del cambio"

# 4. Subir a tu fork
git push origin contribution

# 5. Crear PR desde tu fork hacia el repo original
#    → En GitHub: New Pull Request
#    → base: boinor/boinor (upstream) ← head: JuanJo-Saavedra/boinor:contribution
```

---

## Flujo de Desarrollo Diario (gemelo-digital)

```bash
# Empezar a trabajar
git checkout gemelo-digital

# Hacer cambios, commitear
git add .
git commit -m "feat(dt): agregar modelo de subsistema ADCS"

# Subir al fork
git push origin gemelo-digital
```

---

## Comandos de Referencia Rápida

| Acción | Comando |
|--------|---------|
| Ver en qué rama estoy | `git branch --show-current` |
| Ver todas las ramas | `git branch -a` |
| Ver remotes configurados | `git remote -v` |
| Ver cambios sin commitear | `git status` |
| Ver historial de commits | `git log --oneline --graph --all` |
| Crear nueva rama | `git checkout -b nombre-rama` |
| Cambiar de rama | `git checkout nombre-rama` |
| Traer cambios de upstream | `git fetch upstream` |
| Fusionar rama en la actual | `git merge otra-rama` |
| Subir rama al fork | `git push origin nombre-rama` |
| Ver diferencias entre ramas | `git diff main..gemelo-digital` |
