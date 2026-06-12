# Documentación de boinor

> **Estructura**: Esta carpeta contiene la documentación oficial de boinor (`source/`) y los documentos de análisis del proyecto de gemelo digital (`init/`).

## Estructura

```
docs/
├── source/          # Documentación oficial Sphinx (Read the Docs)
│   ├── api.md       # Referencia de API
│   ├── quickstart.md
│   ├── installation.md
│   ├── gallery.md   # Galería de ejemplos
│   ├── examples/    # Notebooks ejecutables (.myst.md)
│   └── conf.py      # Configuración Sphinx
├── init/            # Análisis y propuesta gemelo digital
│   ├── analisis-boinor-gemelo-digital.md
│   ├── propuesta-trabajo-flujo-git.md
│   ├── filosofia-desarrollo-stack-boinor.md
│   └── oportunidades-contribucion-upstream.md
├── Makefile         # Build system
└── README.md        # Este archivo
```

---

## Documentación Oficial (`source/`)

### Stack Tecnológico

| Componente | Herramienta |
|------------|-------------|
| **Motor** | Sphinx |
| **Tema** | sphinx_rtd_theme (Read the Docs) |
| **Formato** | MyST Markdown + reStructuredText |
| **Notebooks** | nbsphinx + jupytext |
| **API docs** | sphinx-autoapi (auto-generada) |
| **Bibliografía** | sphinxcontrib-bibtex |
| **Matemáticas** | MathJax v2 |

### Extensiones Sphinx (15)

```python
extensions = [
    "autoapi.extension",           # API docs automáticas
    "sphinx.ext.autodoc",          # Docstrings inline
    "sphinx.ext.napoleon",         # Estilo Google/NumPy
    "sphinx.ext.intersphinx",      # Enlaces cruzados (Python, Astropy, NumPy, SciPy)
    "nbsphinx",                    # Jupyter Notebooks
    "sphinx_gallery.load_style",   # Galería de ejemplos
    "myst_parser",                 # Markdown MyST
    "sphinx_copybutton",           # Botón copiar código
    "hoverxref.extension",         # Tooltips interactivos
    "sphinxcontrib.bibtex",        # Bibliografía BibTeX
    # ... y más
]
```

### Organización del Contenido (Diátaxis Framework)

| Sección | Archivos | Propósito |
|---------|----------|-----------|
| **Tutorials** | `installation.md`, `quickstart.md` | Guías paso a paso para empezar |
| **How-to Guides** | `gallery.md` + `examples/*.myst.md` | 19 notebooks prácticos |
| **Reference** | `api.md`, `changelog.md`, `bibliography.md` | API docs, historial, referencias |
| **Background** | `history.md`, `related.md`, `background.md` | Contexto teórico y proyectos relacionados |

### Notebooks de Ejemplo

**Formato**: `.myst.md` (MyST Markdown notebooks)

**Ejemplos destacados**:
- `going-to-mars-with-python-using-poliastro.myst.md` — Misión a Marte
- `plotting-in-3D.myst.md` — Visualización 3D
- `detecting-events.myst.md` — Detección de eventos orbitales
- `loading-OMM-and-TLE-satellite-data.myst.md` — Carga de datos satelitales
- `propagation-using-cowells-formulation.myst.md` — Propagación con perturbaciones

**Conversión**:
```bash
# Convertir .myst.md → .ipynb
jupytext --to ipynb examples/*.myst.md

# Ejecutar notebooks
jupytext --execute examples/*.ipynb
```

### Build Commands

```bash
# Build HTML
make html

# Build PDF (recomendado xelatex para Unicode)
make xelatexpdf

# Convertir y ejecutar notebooks
make examples

# Verificar enlaces externos
make linkcheck

# Limpiar build
make clean

# Saltar ejecución de notebooks (más rápido)
BOINOR_SKIP_NOTEBOOKS=True make html
```

### API Documentation

**Híbrida: Manual + Auto-generada**

- **High-level API** (`Orbit`, `Ephem`, `Maneuver`): Manual con `.. autoapiclass::` para control fino
- **Core API**: Auto-generada vía `sphinx-autoapi` escaneando `src/boinor/core/`
- **Resto de paquetes**: Auto-generada con glob patterns

**Configuración AutoAPI** (`conf.py`):
```python
autoapi_type = "python"
autoapi_dirs = ["../../src/"]
autoapi_options = [
    "members",
    "undoc-members",
    "show-inheritance",
    "show-module-summary",
    "special-members",
    "inherited-members",
]
```

### Assets Estáticos

```
_static/
├── css/custom.css      # Overrides del tema RTD
├── favicon.ico
├── logo_text.png
├── thumbnails/         # Miniaturas para galería
└── *.png, *.gif        # Imágenes de ejemplos (Hohmann, MSL, CZML, etc.)
```

---

## Documentación de Gemelo Digital (`init/`)

Análisis y propuesta para construir un gemelo digital de satélite sobre boinor.

| Archivo | Contenido |
|---------|-----------|
| **analisis-boinor-gemelo-digital.md** | Arquitectura de boinor, clases clave, fortalezas/brechas para DT, roadmap propuesto |
| **propuesta-trabajo-flujo-git.md** | Estrategia de ramas (`gemelo-digital` + `contribution`), flujo de actualización paso a paso |
| **filosofia-desarrollo-stack-boinor.md** | Principios ("physics > PEP8"), stack (numba/astropy/scipy), patrones de implementación |
| **oportunidades-contribucion-upstream.md** | 10 features priorizadas para contribuir al repo original |

### Resumen Ejecutivo

**boinor** es una biblioteca de astrodinámica de alta calidad técnica (8/10 para mecánica orbital). Para un gemelo digital operacional completo necesita:

| Brecha | Prioridad |
|--------|-----------|
| Telemetría real-time (TLE, CCSDS TM, MQTT/Kafka) | 🔴 Crítica |
| Modelos de subsistemas (ADCS, EPS, TTC, Thermal) | 🔴 Crítica |
| Bus de eventos / pub-sub | 🟡 Alta |
| Detección de anomalías y mantenimiento predictivo | 🟡 Alta |
| Fault injection framework | 🟡 Media |

### Estrategia de Ramas

```
upstream (boinor/boinor)
    │
    ▼
origin (tu fork)
    ├── main              ← base estable
    ├── gemelo-digital    ← features del DT (NO van a upstream)
    └── contribution      ← PRs al upstream original
```

---

## Áreas de Mejora Identificadas

### Documentación Oficial (`source/`)

| # | Problema | Solución |
|---|----------|----------|
| 1 | Marca residual "poliastro" en `gallery.md` y notebooks | Renombrar a "boinor" |
| 2 | AutoAPI toctree hack (`[!c_]*/index`) | Estructura explícita o mejorar glob |
| 3 | Solo 8/19 notebooks tienen thumbnail | Completar `nbsphinx_thumbnails` en `conf.py` |
| 4 | `sphinx_rtd_theme.get_html_theme_path()` deprecated | Actualizar a método moderno |
| 5 | MathJax v2 (lento) | Migrar a MathJax v3 |
| 6 | Intersphinx SciPy URL fijo (`scipy-1.8.0`) | Cambiar a `stable/` |
| 7 | `nbsphinx_execute = "always"` (lento en CI) | Usar caché de ejecución |
| 8 | Falta `make.bat` para Windows | Añadir script Windows |
| 9 | Landing page (`index.md`) muy larga | Separar "Getting Started" |
| 10 | BibTeX no referenciado consistentemente | Auditar citas `{cite:p}` |

### Oportunidades de Contribución

**Quick wins** (fácil, alto valor, alta aceptación):
- **A**: Subir cobertura de docstrings (50% → 80%)
- **G**: Mejorar mensajes de error en `cowell.py`

**Ver**: `init/oportunidades-contribucion-upstream.md` para lista completa.

---

## Enlaces

- **Documentación online**: https://boinor.readthedocs.io/
- **Repo original**: https://github.com/boinor/boinor
- **Fork (gemelo digital)**: https://github.com/JuanJo-Saavedra/boinor
- **PyPI**: https://pypi.org/project/boinor/
- **Conda Forge**: https://anaconda.org/conda-forge/boinor
