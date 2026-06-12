# Oportunidades de Contribución al Repo Original boinor

> **Resumen**: 10 features priorizadas para contribuir a upstream, evaluadas por complejidad, valor y probabilidad de aceptación.

## Criterios de Evaluación

| Criterio | Pregunta clave |
|----------|----------------|
| **Filosofía** | ¿Encaja en el enfoque de boinor (astrodinámica general, no ops de satélites)? |
| **Complejidad** | ¿Cuánto esfuerzo requiere? |
| **Valor** | ¿Beneficia a la comunidad? |
| **Aceptación** | ¿Es probable que el mantenedor lo acepte? |

---

## Prioridad 1 — Fácil, Alto Valor, Alta Aceptación

### A. Mejorar cobertura de docstrings

**Qué**: Subir meta de `interrogate` de 50% a 70-80% en `pyproject.toml:222`.

**Por qué**: El mantenedor está activamente limpiando pylint y docstrings (changelog muestra "taking care of pylint issues" en múltiples commits recientes).

**Complejidad**: 🟢 Fácil — añadir docstrings Sphinx a funciones públicas.

**Valor**: 🟢 Alto — mejora documentación API automática.

**Aceptación**: 🟢 Muy alta — alineado con trabajo actual del mantenedor.

**Cómo empezar**:
```bash
# Ver funciones sin docstrings
interrogate src/boinor/twobody/ -v

# Añadir docstrings estilo Sphinx
def propagate(self, tof, method=FarnocchiaPropagator()):
    """Propagate orbit forward or backward in time.
    
    Parameters
    ----------
    tof : ~astropy.units.Quantity
        Time of flight.
    method : Propagator, optional
        Propagation method. Default: FarnocchiaPropagator.
    
    Returns
    -------
    Orbit
        Propagated orbit.
    """
```

---

### B. Añadir type hints a la API pública

**Qué**: Anotaciones `->` y tipos de parámetros en `Orbit`, `Body`, `Ephem`, `Maneuver`.

**Por qué**: mypy ya está en el stack de testing. Mejora DX para usuarios con IDEs modernos.

**Complejidad**: 🟡 Medio — `astropy.units.Quantity` no tiene buen soporte en mypy, usar `Any` o stubs.

**Valor**: 🟢 Alto — autocompletado y detección de errores en IDEs.

**Aceptación**: 🟢 Alta — no rompe compatibilidad.

**Ejemplo**:
```python
from typing import Any

@u.quantity_input(r=u.km, v=u.km/u.s)
def from_vectors(
    cls,
    attractor: Body,
    r: Any,  # astropy.units.Quantity
    v: Any,  # astropy.units.Quantity
    epoch: Time = J2000,
    plane: Planes = Planes.EARTH_EQUATOR,
) -> "Orbit":
    ...
```

---

### C. Tests property-based para funciones de anomalías

**Qué**: `hypothesis` tests para `core/angles.py` (conversiones E↔F↔D↔M↔ν).

**Por qué**: Algoritmos de anomalías son críticos y propensos a errores de branch (parabólicas, hiperbólicas).

**Complejidad**: 🟡 Medio — escribir generadores de hypothesis para rangos válidos.

**Valor**: 🟢 Alto — detecta regresiones en casos edge.

**Aceptación**: 🟢 Alta — ya usan hypothesis en `test_propagation.py`.

**Ejemplo**:
```python
from hypothesis import given, strategies as st

@given(
    ecc=st.floats(min_value=0.0, max_value=0.99),
    M=st.floats(min_value=0.0, max_value=2*np.pi),
)
def test_M_to_E_roundtrip(ecc, M):
    E = M_to_E(ecc, M)
    M_recovered = E_to_M(ecc, E)
    assert np.isclose(M, M_recovered, rtol=1e-10)
```

---

### D. Vectorizar funciones de propagación faltantes

**Qué**: Funciones en `core/angles.py` que aún no aceptan arrays (similar a `coe2rv_many`).

**Por qué**: Rendimiento en batch processing. El patrón ya existe (`coe2rv_many` usa `numba.prange`).

**Complejidad**: 🟡 Medio — añadir versiones `_many` con `@jit(parallel=True)`.

**Valor**: 🟡 Medio — speedup para análisis de constelaciones.

**Aceptación**: 🟢 Alta — patrón establecido.

---

### G. Mejorar mensajes de error en propagación

**Qué**: `cowell.py:27` lanza `RuntimeError("Integration failed")` sin contexto. Añadir info sobre tolerancia, tiempo, eventos.

**Por qué**: Debugging mucho más fácil para usuarios.

**Complejidad**: 🟢 Fácil — añadir parámetros al mensaje de error.

**Valor**: 🟢 Alto — calidad de vida.

**Aceptación**: 🟢 Muy alta — mejora no controversial.

**Ejemplo**:
```python
# Antes:
raise RuntimeError("Integration failed")

# Después:
raise RuntimeError(
    f"Integration failed: tof={tof}s, rtol={rtol}, atol={atol}. "
    f"Try increasing tolerances or reducing time of flight."
)
```

---

## Prioridad 2 — Medio, Buen Valor, Buena Aceptación

### E. Nuevos modelos de perturbación

**Qué**:
- Sombra cilíndrica/cónica para `radiation_pressure` (ahora solo line-of-sight booleano)
- Perturbación por arrastre magnético (magnetic drag)
- Efecto Yarkovsky para asteroides

**Por qué**: Aumenta capacidad para análisis de órbitas LEO y asteroides.

**Complejidad**: 🟠 Medio-Alto — física no trivial, validación contra literatura.

**Valor**: 🟢 Alto — casos de uso reales (debris, asteroides).

**Aceptación**: 🟡 Buena — patrón de composición ya establecido.

**Cómo empezar**:
```python
@jit
def shadow_conical(t0, state, k, R_body, R_sun, r_sun):
    """Conical shadow model (umbra + penumbra).
    
    Returns
    -------
    float
        Shadow factor (0 = full shadow, 1 = full sun).
    """
    # Geometría de cono
    ...
```

---

### F. Propagador de Encke

**Qué**: Propagador que usa elementos osculantes en lugar de integración directa (Cowell). Útil para perturbaciones pequeñas.

**Por qué**: Mejor estabilidad numérica para propagaciones largas con perturbaciones débiles.

**Complejidad**: 🟠 Medio-Alto — implementación no trivial, validación.

**Valor**: 🟡 Medio — nicho pero útil para misiones interplanetarias.

**Aceptación**: 🟡 Buena — ya tienen 9 propagadores, patrón claro.

**Referencia**: Vallado, "Fundamentals of Astrodynamics and Applications", Algorithm 10.

---

### J. Paralelización de `propagate_many`

**Qué**: `FarnocchiaPropagator.propagate_many` usa list comprehension. Podría vectorizarse con numba o multiprocessing.

**Por qué**: Speedup significativo para análisis de constelaciones.

**Complejidad**: 🟡 Medio — cuidado con astropy units (no son numba-compatible).

**Valor**: 🟡 Medio — caso de uso específico.

**Aceptación**: 🟡 Buena — pero requiere discusión de approach.

---

## Prioridad 3 — Difícil, Depende de Discusión

### H. Propagador SGP4/SDP4 nativo

**Qué**: Wrapper nativo de TLE propagation en la API `Orbit`.

**Por qué**: Enorme para comunidad satelital.

**Complejidad**: 🔴 Alto — integración con `sgp4` library, manejo de TLE parsing.

**Valor**: 🟢 Alto — pero ya existen librerías especializadas (`sgp4`, `skyfield`).

**Aceptación**: 🟠 Depende — boinor se enfoca en astrodinámica general, no necesariamente ops de satélites. **Abrir issue primero**.

---

### I. Frame de coordenadas MEE completo

**Qué**: Ya tienen `ModifiedEquinoctialState` en `states.py`. Pero no hay un frame astropy completo para MEE.

**Por qué**: MEE es útil para optimización de trayectorias y control.

**Complejidad**: 🔴 Alto — integración con astropy.coordinates.

**Valor**: 🟡 Medio — nicho.

**Aceptación**: 🟠 Media — necesitaría demostrar caso de uso claro.

---

## Tabla Resumen

| ID | Feature | Complejidad | Valor | Aceptación | Recomendación |
|----|---------|-------------|-------|------------|---------------|
| A | Docstrings | 🟢 Fácil | 🟢 Alto | 🟢 Muy alta | **Empezar aquí** |
| B | Type hints | 🟡 Medio | 🟢 Alto | 🟢 Alta | Buen segundo PR |
| C | Tests anomalías | 🟡 Medio | 🟢 Alto | 🟢 Alta | Si conoces hypothesis |
| D | Vectorización | 🟡 Medio | 🟡 Medio | 🟢 Alta | Si vienes de performance |
| G | Mensajes error | 🟢 Fácil | 🟢 Alto | 🟢 Muy alta | **Quick win** |
| E | Perturbaciones | 🟠 Medio-Alto | 🟢 Alto | 🟡 Buena | Si conoces la física |
| F | Propagador Encke | 🟠 Medio-Alto | 🟡 Medio | 🟡 Buena | Si tienes experiencia |
| J | Paralelización | 🟡 Medio | 🟡 Medio | 🟡 Buena | Discutir approach |
| H | SGP4 nativo | 🔴 Alto | 🟢 Alto | 🟠 Depende | Abrir issue primero |
| I | Frame MEE | 🔴 Alto | 🟡 Medio | 🟠 Media | Demostrar caso de uso |

---

## Estrategia de Contribución Recomendada

### Para tu primer PR

1. **Elige A o G** — fácil, alto valor, alta aceptación
2. **Fork y branch**: ya tienes `contribution` desde `main`
3. **Sigue convenciones**: Black, ruff, tests pytest
4. **PR pequeño**: 1-2 archivos, <100 líneas
5. **Referencia issues**: si hay issues abiertos relacionados, menciónalos

### Para PRs posteriores

1. **B o C** — medio, alto valor
2. **Discute en issue**: para E, F, H, I abre issue antes de implementar
3. **Validación**: compara contra literatura (Vallado, Curtis) o software (GMAT, Orekit)

### Qué NO contribuir

- Features específicas de gemelo digital (telemetría real-time, subsistemas, fault injection) → van en tu rama `gemelo-digital`
- Cambios que rompan la arquitectura de dos capas (core puro vs API con unidades)
- Optimizaciones prematuras sin benchmarks
