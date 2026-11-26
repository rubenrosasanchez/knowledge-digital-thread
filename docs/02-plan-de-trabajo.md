# Plan de trabajo

Este documento reparte el desarrollo de `knowledge-digital-thread` en tareas que se pueden asignar por separado. Parte del alcance descrito en [01-arquitectura.md](01-arquitectura.md) y no lo modifica.

Cada tarea se registra como una issue de GitHub con una etiqueta de tipo (`feature`, `bug`, `docs`, `ci`), una de flujo (A a E) y una de objetivo específico (OE1 a OE6). Las tareas sin dependencias se pueden empezar desde el primer momento. Las demás indican qué tarea necesitan.

## Contratos compartidos

Varios flujos trabajan contra las mismas definiciones, así que conviene fijarlas antes de repartir el resto. Son pocas y pequeñas:

1. Convención de identificadores (`REQ-001`, `PART-Battery`, `TC-07`) y patrón de URIs `https://w3id.org/kdt/id/{tipo}/{ID}`.
2. Modelo de dominio en `src/dt/model.py`: `Requirement`, `Part` y `TestCase`.
3. Borrador de la ontología en `ontology/dt.ttl`, con los términos OSLC y `dt:` que se van a usar.
4. Grafo de ejemplo en `tests/fixtures/grafo-ejemplo.ttl`, pequeño y con huecos de trazabilidad a propósito.
5. Interfaz `LinkSuggester` y firma del cliente de almacén.
6. Esqueleto del repositorio, con las ramas `develop` y `feature/<issue>-<descripcion>`.

El modelo de dominio (D1) y el grafo de ejemplo (C1) son los que más tareas desbloquean. Una vez fusionados en `develop`, cada flujo puede avanzar con el grafo de ejemplo sin esperar a que los demás terminen.

## Flujo A: proceso, documentación y evaluación

| ID | Tarea | Depende de | OE |
|---|---|---|---|
| A1 | Configurar GitHub: Project con sprints, etiquetas, milestones por objetivo específico, rama `develop` y protección de `main` | Ninguna | OE1 |
| A2 | Subir la documentación de arquitectura al repositorio y crear las issues de todos los flujos | A1 | OE1 |
| A3 | Línea base de alcance, plazo y esfuerzo, y registro de riesgos | Ninguna | OE6 |
| A4 | Requisitos del propio proyecto en patrones EARS, trazados a los objetivos específicos | Ninguna | OE2 |
| A5 | Estimación de esfuerzo con COSYSMO, comprobando los pesos en la fuente original | Ninguna | OE6 |
| A6 | Política de uso de IA, `docs/ia-log.md` e inicialización de OpenSpec | Decisión del equipo | OE1 |
| A7 | ADR-001 a ADR-005 en formato MADR | Ninguna | OE1, OE2 |
| A8 | Diseño del experimento de evaluación y de sus métricas | A4, C1 | OE6 |
| A9 | Guion de la demo y ejecución del experimento | Integración I2 | OE6 |

## Flujo B: plataforma, calidad y almacén

| ID | Tarea | Depende de | OE |
|---|---|---|---|
| B1 | `pyproject.toml` con uv, Ruff y pytest | Ninguna | OE4 |
| B2 | Workflow de CI: Ruff, tests con cobertura y radon | B1 | OE4 |
| B3 | Dockerfile de Fuseki con versión fijada, `config.ttl` (TDB2) y `docker-compose.yml` | Ninguna | OE4 |
| B4 | Cliente de Fuseki: carga por Graph Store Protocol y consultas SPARQL | B3, C1 | OE4 |
| B5 | MkDocs y script que genera la matriz de trazabilidad y la VCRM | B1 | OE6 |
| B6 | `Makefile` con el objetivo `make all` | B2, B3 | OE4 |
| B7 | Workflows de publicación de la documentación y de las imágenes en GHCR | B5, B6 | OE4 |

## Flujo C: semántica y datos

| ID | Tarea | Depende de | OE |
|---|---|---|---|
| C1 | `ontology/dt.ttl` y `tests/fixtures/grafo-ejemplo.ttl` | Ninguna | OE2 |
| C2 | Caso de estudio del dron en ReqIF, SysML v2 y Gherkin, con `data/ground-truth.csv` | Ninguna | OE6 |
| C3 | Documentar los defectos inyectados en el caso de estudio | C2 | OE6 |
| C4 | `shapes/trazabilidad.ttl` con las tres shapes y sus tests | C1 | OE4 |
| C5 | Consultas Q1 a Q6 en ficheros `.rq`, con tests sobre el grafo de ejemplo | C1 | OE4 |

## Flujo D: ingesta

| ID | Tarea | Depende de | OE |
|---|---|---|---|
| D1 | `model.py` con las dataclasses y la convención de identificadores | Ninguna | OE2 |
| D2 | Parser de ReqIF con tests unitarios | D1 | OE3 |
| D3 | Gramática Lark para el subconjunto de SysML v2, con tests y ADR-003 | D1 | OE3 |
| D4 | Parser de Gherkin con tests unitarios | D1 | OE3 |
| D5 | Mapeo de los objetos de dominio a RDF | D1, C1 | OE3 |
| D6 | Resolución de enlaces explícitos, con avisos para los identificadores que no existen | D5 | OE4 |
| D7 | CLI con Typer: `ingest`, `validate`, `trace` e `impact` | D6, B4, C4, C5 | OE4 |

Hasta que C2 publique el caso de estudio, los parsers se prueban con ficheros de muestra propios en `tests/fixtures`.

## Flujo E: IA e interfaz

| ID | Tarea | Depende de | OE |
|---|---|---|---|
| E1 | Prueba de concepto del Copilot SDK con las cuentas del equipo | Ninguna | OE4 |
| E2 | Interfaz `LinkSuggester`, doble de pruebas con respuestas grabadas y esquema JSON de la respuesta | D1 | OE4 |
| E3 | Sugerencias de enlaces con el Copilot SDK y generación de `data/suggestions.ttl` | E1, E2, C2 | OE4 |
| E4 | Comando `dt suggest` | E3, D7 | OE4 |
| E5 | Pantalla del explorador en Streamlit con `st-cytoscape`, sobre el grafo de ejemplo | C1 | OE5 |
| E6 | Pantallas de cobertura y de impacto | E5, C4, C5 | OE5 |
| E7 | Aceptar o rechazar sugerencias y copiar las aceptadas al grafo `main` | E3, E5 | OE5 |

Si el Copilot SDK no funciona con las credenciales disponibles, el sistema carga `data/suggestions.ttl` y las tareas E3 y E4 se adaptan a ese caso.

## Puntos de integración

| Hito | Qué debe funcionar |
|---|---|
| I1 | Recorrido completo con enlaces explícitos: ficheros fuente, RDF, carga en Fuseki, validación SHACL, consultas y CLI, con el CI en verde |
| I2 | Interfaz sobre el grafo real, sugerencias de IA aprobables desde la interfaz y tests funcionales contra Fuseki |
| Cierre | `make all` en una máquina limpia, con documentación generada y evaluación ejecutada |

## Convenciones

- Cada tarea tiene su issue y su rama `feature/<issue>-<descripcion>` creada desde `develop`.
- A `main` solo se llega por pull request con el CI en verde. No se fusiona código sin tests.
- Cada funcionalidad grande (parsers, enlazado con IA, validación e interfaz) empieza con una propuesta en `openspec/`, escrita por quien la implementa.
- Cada pull request indica qué parte se hizo con IA, y se anota en `docs/ia-log.md`.
- Los tests unitarios no llaman al LLM ni a Fuseki. Usan `rdflib` en memoria y el doble de `LinkSuggester`.
- Los flujos son dueños de sus carpetas para reducir conflictos al fusionar: A en `docs/` y `openspec/`, B en `docker/`, `.github/`, `Makefile` y `src/dt/store/`, C en `ontology/`, `shapes/`, `data/`, `src/dt/validation/` y `src/dt/queries/`, D en `src/dt/ingest/`, `src/dt/mapping/` y `src/dt/cli.py`, E en `src/dt/linking/` y `src/dt/ui/`.
