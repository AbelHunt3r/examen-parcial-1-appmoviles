# Examen Parcial 1 — Pull Request con aseguramiento de calidad

**Desarrollo de aplicaciones móviles nativas · Grupo 7CV4 · IPN-ESCOM**

## Equipo

- **Autor de esta entrega:** Reyes Castellanos José Abel — boleta 2020311353 — GitHub: [@AbelHunt3r](https://github.com/AbelHunt3r)
- Compañeros de equipo (para revisión entre pares): Chávez Romero Jonathan (2024630102), Moreno Noguerón Ximena (2024630201)

Cada integrante del equipo entregó su propio bug/mejora de forma individual sobre su propio fork,
según lo pedido en el examen. Este repositorio documenta únicamente la entrega de José Abel.

## Objetivo

Elegir, corregir y documentar con aseguramiento de calidad (QA) un bug pequeño y real en
[`gabrielhuav/PolitecnicoOpenWorld`](https://github.com/gabrielhuav/PolitecnicoOpenWorld), siguiendo
un flujo completo de control de versiones (fork, issue, rama, commits incrementales, Draft PR) y
una matriz de pruebas manual de al menos 6 casos.

## Alcance

**Bug corregido:** el buscador de estaciones de Metro/Metrobús (menú de Teletransporte, Modo
Desarrollador) no encontraba nombres de estación con acento cuando se buscaban sin acento (p. ej.
"pantitlan" no encontraba "Pantitlán"). Se corrigió comparando ambos lados normalizados (sin
acentos, preservando la ñ como letra propia del español).

**Fuera de alcance:** cualquier otro buscador o filtro de texto del proyecto que pudiera compartir
el mismo patrón — no se auditó el resto del código en busca de instancias similares.

## Issue

[`AbelHunt3r#1` — Station search doesn't match accented names when typed without accents](https://github.com/AbelHunt3r/PolitecnicoOpenWorld/issues/1)

## Pull Request

`[COMPLETAR — link al Draft PR una vez abierto hacia gabrielhuav/PolitecnicoOpenWorld:main]`

## SHAs

- **SHA base** (main del original, antes del cambio): `7ed325393f82872c2be94ff2ada46948efa19152`
- **SHA final entregado:** `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2`
- Los commits posteriores a este SHA, si los hay, solo tocan documentación (no afectan el código
  probado) — `[CONFIRMAR al momento de la entrega final]`.

## Matriz de pruebas

Ver [`docs/pruebas.md`](docs/pruebas.md) — 6 casos ejecutados (ruta feliz, límite, regresión,
navegación/estado, accesibilidad, compatibilidad), todos aprobados, sin defectos encontrados.

## Evidencia

Capturas y evidencia de cada caso: `[COMPLETAR — agregar carpeta evidencia/ con las capturas y
enlazarlas aquí y desde cada caso de docs/pruebas.md]`.

## Checks de CI (PR Quality Gate)

`[COMPLETAR una vez abierto el PR — estado de cada check (build Android debug, unit tests
app/shared, chequeo de nombres de test KMP, detekt), SHA evaluado, y link a la pestaña Checks del
PR]`.

## Revisión entre pares

`[COMPLETAR — quién revisó este PR (Jonathan o Ximena), link a su revisión, y cómo se
respondieron sus observaciones]`.

## Conclusiones

El cambio es pequeño y aislado: un archivo nuevo (`StationSearch.kt`) con una función pura y
testeable, más el reemplazo de dos comparaciones existentes en `WorldMapScreen.kt`. No toca
persistencia, red ni lógica de juego. Las 6 pruebas manuales y los 7 unit tests nuevos confirman
que corrige el bug sin romper funcionalidad existente. Se recomienda su integración.

## Bitácora (José Abel Reyes Castellanos)

- Sincronización del fork con `upstream/main`, SHA base registrado: `7ed32539...`.
- Reproducción del bug en la versión base (buscador sin resultados al buscar "pantitlan").
- 3 commits incrementales en la rama `fix/station-search-accents`:
  1. `fix: ignore accents when searching metro/metrobus stations` — archivo nuevo `StationSearch.kt`.
  2. `test: add unit tests for accent-insensitive station search` — 7 casos en `StationSearchTest.kt`.
  3. `fix: use stationMatchesQuery in the metro/metrobus search filters` — integración en `WorldMapScreen.kt`.
- Ejecución de los 6 casos de `docs/pruebas.md` en emulador (Medium Phone, API 37.1) — todos aprobados.
- Compilación y suite completa de tests corridos en local (`BUILD SUCCESSFUL`, 79 tareas).
- `[COMPLETAR: link a la revisión que diste a un PR de un compañero, una vez hecha]`.

## Herramientas de IA utilizadas

Se usó **Claude (Cowork)** como asistente durante toda la entrega, para:
- Explorar el código del repositorio y encontrar un candidato de bug real, pequeño y verificable.
- Redactar el código de la corrección (`StationSearch.kt`) y sus pruebas unitarias
  (`StationSearchTest.kt`), revisados y entendidos por el autor antes de integrarlos.
- Guiar paso a paso el flujo de Git/GitHub (fork, sync, ramas, commits, Draft PR) y resolver
  problemas de entorno encontrados en el camino (p. ej. `gradle-wrapper.jar` ausente).
- Ayudar a estructurar la matriz de pruebas (`docs/pruebas.md`) y este README índice, a partir de
  los resultados reales que el autor ejecutó y reportó en el emulador.

Todas las pruebas, capturas y resultados documentados en `docs/pruebas.md` fueron ejecutados
directamente por el autor en su propio entorno (emulador Android Studio); ninguna evidencia fue
generada o simulada por la IA.

## Referencias

- Repositorio original: [gabrielhuav/PolitecnicoOpenWorld](https://github.com/gabrielhuav/PolitecnicoOpenWorld)
- Fork del autor: [AbelHunt3r/PolitecnicoOpenWorld](https://github.com/AbelHunt3r/PolitecnicoOpenWorld)
