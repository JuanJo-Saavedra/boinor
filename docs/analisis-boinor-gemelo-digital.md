# Análisis de boinor — Potencial para Gemelo Digital de Satélite

> **Veredicto**: boinor es una base matemática sólida (8/10) para la capa orbital de un gemelo digital. Para un DT operacional completo necesita una capa de aplicación significativa encima.

## ¿Qué es boinor?

**BOdies IN ORbit** — biblioteca Python de astrodinámica y mecánica orbital. Continuación de poliastro, mantenida por Thorsten Alteholz. Licencia MIT.

- **Repo original**: https://github.com/boinor/boinor
- **Stack**: Python 3.10–3.13, Numba, Astropy, SciPy, Matplotlib/Plotly, Vispy, SPICE
- **Build**: flit_core, tox, ruff, black, mypy, CircleCI

---

## Arquitectura

```
┌──────────────────────────────────────────────┐
│  Visualización                               │
│  plotting (matplotlib/plotly)                │
│  render (Vispy/OpenGL 3D)                    │
│  czml (exportación a CesiumJS)               │
├──────────────────────────────────────────────┤
│  API de Dominio                              │
│  Orbit · Body · Maneuver · Spacecraft        │
│  EarthSatellite · Ephem · Sensors            │
├──────────────────────────────────────────────┤
│  Mecánica Orbital                            │
│  twobody · threebody · frames · earth        │
├──────────────────────────────────────────────┤
│  Cálculo Numérico (Numba JIT)                │
│  core: elements · propagation · perturbations│
│        angles · events                       │
├──────────────────────────────────────────────┤
│  Infraestructura Matemática                  │
│  _math: ivp · linalg · optimize · interpolate│
└──────────────────────────────────────────────┘
```

**Principio clave**: `core/` nunca importa `astropy.units`. La capa numérica es pura (arrays + numba). Las unidades físicas viven solo en la API de dominio.

---

## Clases Principales

| Clase | Archivo | Responsabilidad |
|-------|---------|-----------------|
| `Orbit` | `twobody/orbit/scalar.py` | Objeto central: estado + época + propagación/maniobras |
| `BaseState` | `twobody/states.py` | Jerarquía: `ClassicalState`, `RVState`, `ModifiedEquinoctialState` |
| `Body` | `bodies.py` | Cuerpo celeste inmutable (namedtuple con parámetros físicos) |
| `Ephem` | `ephem.py` | Efemérides con interpolación spline/sinc |
| `Maneuver` | `maneuver.py` | Hohmann, bi-elíptica, Lambert |
| `Spacecraft` | `spacecraft/__init__.py` | Modelo básico: área, Cd, masa |
| `EarthSatellite` | `earth/__init__.py` | Órbita terrestre con perturbaciones J2 + atmósfera |
| `CowellPropagator` | `twobody/propagation/` | Integración numérica DOP853, perturbaciones arbitrarias |
| `OrbitPlotter` | `plotting/orbit/plotter.py` | Visualización con backends intercambiables |
| `CZMLExtractor` | `czml/extract_czml.py` | Exportación a CesiumJS |

---

## Patrones de Diseño

- **Strategy** → 9 propagadores intercambiables (`CowellPropagator`, `FarnocchiaPropagator`, `ValladoPropagator`, etc.)
- **State** → `BaseState` con 3 representaciones convertibles entre sí
- **Mixin** → `OrbitCreationMixin` separa factory methods de la clase `Orbit`
- **Template/Backend** → `OrbitPlotter` delega dibujo a backend (`Matplotlib2D`, `Plotly2D`)
- **`@u.quantity_input`** → validación de unidades físicas en toda la API pública
- **Import Linter** → garantiza que `core` no dependa de `astropy.units`

---

## Testing

| Tipo | Herramienta | Ejemplo |
|------|-------------|---------|
| Unitarios | pytest | `test_bodies.py`, `test_maneuver.py` |
| Propagación | pytest + parametrize | Todos los propagadores con órbitas conocidas |
| Property-based | hypothesis | Tiempos de vuelo aleatorios preservan elementos orbitales |
| Benchmarks | pytest-benchmark | Comparación de rendimiento entre implementaciones |
| Imagen | pytest-mpl | Comparación de plots generados |
| Validación externa | workflow GitHub | Comparación contra GMAT y Orekit |
| CI/CD | CircleCI | Multi-Python (3.10–3.14), fast/slow/images/coverage |

---

## Evaluación para Gemelo Digital

### ✅ Fortalezas (lo que ya existe)

| Capacidad | Madurez | Detalle |
|-----------|---------|---------|
| Propagación orbital | 🟢 Alta | 9 propagadores validados contra Vallado/Curtis |
| Perturbaciones | 🟢 Media-Alta | J2, J3, arrastre, presión radiación, tercer cuerpo |
| Maniobras | 🟢 Alta | Hohmann, bi-elíptica, Lambert, corrección pericentro |
| Efemérides | 🟢 Alta | JPL Horizons, SPICE, interpolación spline/sinc |
| Visualización 3D | 🟢 Funcional | Vispy/OpenGL + CZML para CesiumJS |
| Unidades físicas | 🟢 Excelente | astropy.units en toda la API |
| Modelos atmosféricos | 🟢 Funcional | COESA62, COESA76 |

### ❌ Brechas (lo que falta para un DT completo)

| Brecha | Prioridad | Por qué importa |
|--------|-----------|-----------------|
| **Telemetría real-time** | 🔴 Crítica | Sin ingesta de streams (TLE, CCSDS TM, MQTT/Kafka) no hay DT operacional |
| **Modelos de subsistemas** | 🔴 Crítica | `Spacecraft` solo tiene A, Cd, m. No hay ADCS, EPS, TTC, Thermal |
| **Bus de eventos / pub-sub** | 🟡 Alta | Sin patrón Observer, los componentes no se comunican asíncronamente |
| **Detección de anomalías** | 🟡 Alta | Sin ML/stats sobre telemetría no hay mantenimiento predictivo |
| **Fault injection** | 🟡 Media | Sin simulación de fallos no se prueban escenarios de contingencia |
| **API REST/gRPC** | 🟡 Media | Sin servicio externo no hay composición de escenarios ni integración |
| **Paralelización** | 🟢 Baja | Numba acelera funciones, pero no hay paralelismo a nivel de constelación |

---

## Roadmap Propuesto para el Gemelo Digital

```
Fase 1 — Fundamentos del DT
├── Modelo de subsistemas (ADCS, EPS, TTC, Thermal)
├── Ingesta de telemetría (TLE parser + adapter pattern)
└── Bus de eventos interno

Fase 2 — Simulación Avanzada
├── Fault injection framework
├── Modelos de degradación (paneles, baterías, thrusters)
└── Escenarios multi-satélite

Fase 3 — Operaciones
├── API REST/gRPC para simulaciones remotas
├── Detección de anomalías (ML sobre series temporales)
└── Dashboard de estado del satélite
```
