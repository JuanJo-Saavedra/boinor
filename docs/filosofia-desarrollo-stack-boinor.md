# Filosofía de Desarrollo y Stack de boinor

> **Resumen**: boinor prioriza corrección física sobre estilo, rendimiento sin sacrificar legibilidad, y una arquitectura de dos capas (numérica pura + API con unidades).

## Principios Fundamentales

### 1. "Physics is more important than PEP8"

El changelog lo dice explícitamente. Cuando hay conflicto entre convención de nombres física (μ, ν, ω) y PEP8, gana la física.

### 2. Rendimiento sin sacrificar legibilidad

> *"Can we make Python fast without sacrificing readability?"* — EuroSciPy 2019

**Estrategia**: Numba JIT en la capa numérica pura (`core/`), Python legible en la API (`twobody/`).

### 3. Separación estricta de capas

```
┌─────────────────────────────────────────────────┐
│  twobody/ — API pública                         │
│  ✅ astropy.units, astropy.time                 │
│  ✅ @u.quantity_input para validación           │
│  ❌ NO numba                                    │
├─────────────────────────────────────────────────┤
│  core/ — Cálculo numérico                       │
│  ✅ numba @jit                                  │
│  ✅ numpy arrays puros                          │
│  ❌ NO astropy.units (prohibido por linter)     │
└─────────────────────────────────────────────────┘
```

**Import Linter** (`pyproject.toml:189-199`) prohíbe que `boinor.core` importe `astropy.units`. Esto mantiene la capa numérica compilable por numba sin overhead de objetos Python.

---

## Stack Tecnológico

### Núcleo Numérico

| Librería | Uso | Dónde |
|----------|-----|-------|
| **Numba** | JIT compilation para funciones puras | `core/elements.py`, `core/propagation/`, `core/perturbations.py` |
| **NumPy** | Arrays base, operaciones vectorizadas | Todo `core/` |
| **SciPy** | Integración (DOP853), optimización, interpolación | `_math/ivp.py`, `_math/optimize.py` |

**Patrón numba**:
```python
from numba import njit as jit

@jit
def coe2rv(k, p, ecc, inc, raan, argp, nu):
    # Matemática pura, sin objetos astropy
    ...
```

### Unidades y Coordenadas

| Librería | Uso | Dónde |
|----------|-----|-------|
| **astropy.units** | Unidades físicas, validación | Toda la API pública |
| **astropy.time** | Épocas, escalas de tiempo (UTC, TDB) | `Orbit.epoch`, `Ephem.epochs` |
| **astropy.coordinates** | Frames (ICRS, ITRS), representaciones | `frames/`, `ephem.py` |

**Patrón de validación**:
```python
@u.quantity_input(r=u.km, v=u.km/u.s)
def from_vectors(cls, attractor, r, v, epoch=J2000):
    # r y v ya están validados como unidades de distancia/velocidad
    ...
```

### Visualización

| Backend | Cuándo usar | Archivo |
|---------|-------------|---------|
| **Matplotlib2D** | Scripts, batch processing, papers | `plotting/orbit/backends/matplotlib.py` |
| **Plotly2D/3D** | Jupyter, dashboards interactivos | `plotting/orbit/backends/plotly.py` |
| **Vispy/OpenGL** | Renderizado 3D de shape models (asteroides) | `render/scene.py` |
| **CZML** | Exportación a CesiumJS (web 3D) | `czml/extract_czml.py` |

### Efemérides y Datos

| Librería | Uso |
|----------|-----|
| **spiceypy** | Kernels SPICE (posiciones planetarias precisas) |
| **jplephem** | Efemérides JPL DE430/DE440 |
| **astroquery** | Consultas a JPL Horizons, SBDB (small bodies) |

---

## Convenciones de Código

### Formato y Linting

```toml
# pyproject.toml
[tool.black]
line-length = 79

[tool.ruff]
select = ["E", "F", "I", "NPY201", "NPY003"]
ignore = ["E501"]  # Black maneja longitud de línea

[tool.isort]
profile = "black"
```

### Testing

| Herramienta | Propósito |
|-------------|-----------|
| **pytest** | Framework principal |
| **pytest-cov** | Cobertura (meta: 97%+) |
| **hypothesis** | Property-based testing (invariantes orbitales) |
| **pytest-benchmark** | Comparación de rendimiento |
| **pytest-mpl** | Comparación de imágenes de plots |
| **pytest-doctestplus** | Doctests en docstrings |

**Marcadores**:
- `@pytest.mark.slow` — tests de propagación larga
- `@pytest.mark.remote_data` — consultas a JPL (requieren internet)
- `@pytest.mark.mpl_image_compare` — comparación visual

### Type Hints

**Estado actual**: Casi ausentes. mypy está configurado pero permisivo:
```toml
[tool.mypy]
check_untyped_defs = true
strict_optional = false
ignore_missing_imports = true
```

**Por qué**: `astropy.units.Quantity` no tiene buen soporte en mypy. Prefieren validación en runtime con `@u.quantity_input`.

### Documentación

- **Docstrings**: Estilo Sphinx (no Google)
- **Meta interrogate**: 50% (baja, pero mejorando)
- **Sphinx**: autoapi, sphinx-gallery, myst-parser, nbsphinx
- **Ejemplos**: Notebooks en `docs/source/examples/`

---

## Patrones de Implementación

### Cómo añadir un nuevo propagador

1. **Capa numérica** (`core/propagation/nuevo.py`):
```python
from numba import njit as jit

@jit
def nuevo_propagate_coe(k, p, ecc, inc, raan, argp, nu, tof):
    # Matemática pura
    return p_new, ecc_new, inc_new, raan_new, argp_new, nu_new
```

2. **API pública** (`twobody/propagation/nuevo.py`):
```python
class NuevoPropagator:
    kind = PropagatorKind.ELLIPTIC
    
    def propagate(self, state, tof):
        # Convertir unidades, llamar a core, devolver ClassicalState
        k = state.attractor.k.to(u.km**3 / u.s**2).value
        # ... llamar a nuevo_propagate_coe ...
        return ClassicalState(state.attractor, elements, state.plane)
```

### Cómo añadir una perturbación

**Modelo de composición** (`core/perturbations.py`):
```python
@jit
def J2_perturbation(t0, state, k, J2, R):
    # Devuelve [ax, ay, az] en km/s^2
    ...

# Uso en test_perturbations.py:
def f(t0, u_, k):
    du_kep = func_twobody(t0, u_, k)
    ax, ay, az = J2_perturbation(t0, u_, k, J2=Earth.J2.value, R=Earth.R.to(u.km).value)
    du_ad = np.array([0, 0, 0, ax, ay, az])
    return du_kep + du_ad

# Pasar a Cowell:
orbit.propagate(tof, method=CowellPropagator(f=f))
```

### Cómo definir un nuevo cuerpo

```python
from boinor import Body
from boinor.constants import GM_moon

Moon = Body(
    parent=Earth,
    k=GM_moon,
    name="Moon",
    symbol="☾",
    R=1737.4 * u.km,
    # ... otros parámetros
)
```

---

## Manejo de Errores

| Tipo | Cuándo | Ejemplo |
|------|--------|---------|
| `ValueError` | Parámetros inválidos | Excentricidad negativa |
| `RuntimeError` | Fallos de integración numérica | `solve_ivp` no converge |
| `NotImplementedError` | Funcionalidad no soportada | Propagador no soporta hiperbólicas |
| `warnings.warn` | Casos edge no fatales | `PatchedConicsWarning`, `TimeScaleWarning` |

**Área de mejora**: `cowell.py:27` lanza `RuntimeError("Integration failed")` sin contexto sobre tolerancia, tiempo, o eventos.
