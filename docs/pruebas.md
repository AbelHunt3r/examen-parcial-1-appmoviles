# Matriz de pruebas — Buscador de estaciones sin distinguir acentos

> Repositorio: fork de `gabrielhuav/PolitecnicoOpenWorld` · Rama: `fix/station-search-accents`
> Autor: Reyes Castellanos José Abel — boleta 2020311353 — Grupo 7CV4
> SHA base (main del original, antes del cambio): `7ed325393f82872c2be94ff2ada46948efa19152`
> SHA probado / SHA final entregado: `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2`
> Dispositivo usado en todas las pruebas: Emulador Android Studio — "Medium Phone" (AVD),
> Android 17.0 "CinnamonBun", API 37.1, arm64.
> Fecha de ejecución: 1 de octubre de 2026.

## Cambio bajo prueba

**Dónde:** menú de **Teletransporte** del mapa exterior (solo visible con **Modo Desarrollador**
activo, en Ajustes → Interfaz) → secciones "Estaciones de Metro" / "Estaciones de Metrobús".

**Antes:** el buscador comparaba con `nombre.contains(texto, ignoreCase = true)`. `ignoreCase`
solo ignora mayúsculas/minúsculas, **no acentos**. Del catálogo real (`metro.json`), **54 de las
163 estaciones** llevan acento o ñ (Pantitlán, Juárez, Coyoacán, Zócalo, Instituto del Petróleo,
Martín Carrera…). Buscar `pantitlan` o `juarez` sin acento —lo más común al escribir rápido en un
celular— no encontraba nada, aunque la estación exista. Confirmado en la versión base
(`7ed32539...`) antes de tocar código.

**Después:** se agregó `stationMatchesQuery` (archivo nuevo
`app/src/main/java/.../features/map_exterior/util/StationSearch.kt`), que normaliza acentos en
ambos lados (nombre de estación y texto buscado) antes de comparar. **La ñ se conserva como letra
propia del español** (no se trata como una "n con acento"): buscar `penon` NO debe encontrar
"Peñón", pero buscar `peñon` sí. Se reemplazaron las dos comparaciones (Metro y Metrobús) en
`WorldMapScreen.kt` para usar esta función.

**Usuario afectado:** cualquier jugador que use el buscador de estaciones para teletransportarse
(función de Modo Desarrollador) y escriba sin acentos, como hace la mayoría en un teclado táctil.

**Archivos modificados/nuevos:**
`app/src/main/java/ovh/gabrielhuav/pow/features/map_exterior/util/StationSearch.kt` (nuevo),
`app/src/test/java/ovh/gabrielhuav/pow/features/map_exterior/util/StationSearchTest.kt` (nuevo,
7 unit tests — corre en `:app:testDebugUnitTest`, el mismo gate que usa CI),
`app/src/main/java/ovh/gabrielhuav/pow/features/map_exterior/ui/WorldMapScreen.kt` (2 bloques de
filtro + 1 import).

**Fuera de alcance:** cualquier otro buscador o filtro de texto del proyecto que no sea el de
estaciones de Metro/Metrobús (no se auditó el resto del proyecto en busca del mismo patrón).

**Nota de entorno (no es un defecto de la app):** `./gradlew` no corre de inmediato en una
instalación local limpia porque `gradle/wrapper/gradle-wrapper.jar` está intencionalmente excluido
del repo vía `.gitignore` — el CI del profe no lo necesita porque usa
`gradle/actions/setup-gradle` con `gradle-version: wrapper`, que provisiona Gradle directo. En
local se resolvió instalando Gradle con Homebrew (`brew install gradle`, v9.8.0) y corriendo
`gradle :app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest --stacktrace`
directamente (el mismo comando que ejecuta el workflow, sin el `./`). Preexistente en el repo, no
introducido por este cambio.

## Criterios de aceptación

1. **Éxito:** buscar el nombre de una estación con acento, escrito SIN el acento (p. ej.
   `pantitlan`), la encuentra en los resultados.
2. **Alterno/límite:** una búsqueda vacía sigue mostrando la lista completa; una búsqueda que no
   coincide con ninguna estación real muestra una lista vacía sin crashear; la ñ no se confunde
   con la letra "n".

## Riesgos identificados

| # | Riesgo | Impacto | Caso que lo cubre |
|---|---|---|---|
| 1 | Tratar la ñ como si fuera un acento de la "n" rompería resultados reales (una búsqueda "penon" encontraría "Peñón" incorrectamente). | Medio: resultados de búsqueda incorrectos/confusos, específico del español. | LIM-02 + unit test dedicado en `StationSearchTest` |
| 2 | El cambio de comparador podría romper búsquedas que YA funcionaban bien (con el acento correcto) o las búsquedas parciales a mitad de palabra. | Alto: regresión de una función que antes sí funcionaba. | REG-03 |
| 3 | Aplicar `Normalizer` + regex en cada tecleo sobre ~163+80 estaciones podría sentirse lento en un dispositivo de gama baja. | Bajo: Compose ya usa `remember(query, ...)`, solo recalcula cuando cambia el texto, no en cada frame. | NAV-04 (se observa fluidez al escribir) |

## Matriz de casos (mínimo 6)

### RF-01 — Ruta feliz: buscar sin acento encuentra la estación

- **Criterio cubierto:** criterio de aceptación 1.
- **Autor / fecha:** José Abel Reyes Castellanos — 1 de octubre de 2026.
- **SHA probado / versión de la app:** `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2` / debug
- **Dispositivo/API:** Medium Phone (AVD), Android 17.0, API 37.1, arm64.
- **Precondiciones y datos:** Modo Desarrollador activo (Ajustes → Interfaz). App recién
  instalada desde la compilación de la rama.
- **Pasos:**
  1. Entrar al mapa exterior → abrir menú de opciones (FAB) → "Teletransportarse".
  2. Desplegar "🚇 Estaciones de Metro".
  3. Escribir `pantitlan` en el buscador (sin acento).
  4. Confirmar que "Pantitlán" aparece en la lista filtrada.
  5. Tocarla y confirmar que el personaje se teletransporta ahí.
  6. Repetir con `juarez` → "Juárez" y `coyoacan` → "Coyoacán".
  7. Repetir el mismo flujo en "🚌 Estaciones de Metrobús" con alguna estación con acento del
     catálogo de metrobús.
- **Resultado esperado:** las 3 búsquedas sin acento encuentran su estación real; tocar el
  resultado teletransporta correctamente.
- **Resultado real:** las 3 búsquedas (Pantitlán, Juárez, Coyoacán) encontraron su estación
  correctamente sin escribir el acento; tocar el resultado teletransportó al personaje. El mismo
  comportamiento se repitió en la sección de Metrobús.
- **Estado:** aprobado.
- **Evidencia:**
  Antes (SHA base `7ed32539...`): ![antes](https://github.com/user-attachments/assets/dbe95225-1096-4d1a-9333-4caea52d8bed)
  Después (SHA probado `1b92fed6...`): ![despues](https://github.com/user-attachments/assets/64685653-886d-4bc3-a1f7-91654bc9cb71)
- **Defecto asociado / decisión:** ninguno.

### LIM-02 — Límite: búsqueda vacía, sin coincidencias, y la ñ no se confunde con "n"

- **Criterio cubierto:** criterio de aceptación 2; riesgo #1.
- **Autor / fecha:** José Abel Reyes Castellanos — 1 de octubre de 2026.
- **SHA probado / versión:** `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2` / debug
- **Dispositivo/API:** Medium Phone (AVD), Android 17.0, API 37.1, arm64.
- **Precondiciones y datos:** Modo Desarrollador activo, menú de Teletransporte abierto.
- **Pasos:**
  1. Abrir la sección de Metro sin escribir nada en el buscador — confirmar que se ve la lista
     completa (~163 estaciones).
  2. Escribir `xyz123` (no existe ninguna estación así) — confirmar que la lista queda vacía, sin
     cerrar la app ni mostrar error.
  3. Borrar y escribir solo espacios (`"   "`) — confirmar que vuelve a mostrar la lista completa.
  4. Buscar una estación con ñ con `n` normal en vez de `ñ`, y confirmar que NO aparece.
- **Resultado esperado:** ningún crash en ningún caso; vacío/todo se comporta igual que la versión
  anterior; la ñ se sigue distinguiendo de la "n".
- **Resultado real:** búsqueda vacía mostró la lista completa; "xyz123" dejó la lista vacía sin
  errores ni cierre de la app; solo espacios volvió a mostrar la lista completa. La distinción de
  la ñ se validó adicionalmente con los 2 casos dedicados en `StationSearchTest`
  (`la enie NO se trata como un acento`), que pasaron en la corrida de `testDebugUnitTest`.
- **Estado:** aprobado.
- **Evidencia:** captura de la lista vacía al buscar "xyz123"; salida en verde de
  `StationSearchTest` en la corrida de Gradle (ver REG-03).
- **Defecto asociado / decisión:** ninguno.

### REG-03 — Regresión: búsquedas que ya funcionaban (con acento correcto) siguen igual

- **Criterio cubierto:** riesgo #2.
- **Autor / fecha:** José Abel Reyes Castellanos — 1 de octubre de 2026.
- **SHA probado / versión:** `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2` / debug
- **Dispositivo/API:** Medium Phone (AVD), Android 17.0, API 37.1, arm64 (compilación y tests
  corridos en terminal con Gradle 9.8.0 vía Homebrew — ver nota de entorno arriba).
- **Precondiciones y datos:** N/A.
- **Pasos:**
  1. Buscar `Pantitlán` escribiendo el acento correctamente — confirmar que sigue encontrándola.
  2. Buscar una palabra parcial a mitad de nombre, `tenochtitlan` dentro de
     "Zócalo - Tenochtitlán" — confirmar que la encuentra.
  3. Confirmar que el botón "Ir a tu Ubicación (GPS)" y las demás opciones del menú de
     Teletransporte (no tocadas por este cambio) siguen funcionando igual.
  4. Correr `gradle :app:assembleDebug :app:testDebugUnitTest :shared:testAndroidHostTest --stacktrace`.
- **Resultado esperado:** nada de lo que ya funcionaba se rompió; los tests unitarios pasan.
- **Resultado real:** las búsquedas con acento correcto y la búsqueda parcial siguieron
  funcionando sin cambios; el resto del menú de Teletransporte no mostró diferencias. La
  compilación y los tests terminaron en `BUILD SUCCESSFUL` (79 tareas ejecutadas, ~1m 43s),
  incluidos los 7 casos de `StationSearchTest`.
- **Estado:** aprobado.
- **Evidencia:** captura de terminal con `BUILD SUCCESSFUL`.
- **Defecto asociado / decisión:** ninguno.

### NAV-04 — Navegación y estado: abrir/cerrar el diálogo, colapsar secciones, rotación

- **Criterio cubierto:** estabilidad general del control bajo cambios de estado.
- **Autor / fecha:** José Abel Reyes Castellanos — 1 de octubre de 2026.
- **SHA probado / versión:** `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2` / debug
- **Dispositivo/API:** Medium Phone (AVD), Android 17.0, API 37.1, arm64.
- **Precondiciones y datos:** la actividad principal no fija `screenOrientation` en el manifest,
  así que la rotación es una transición de ciclo de vida válida para este caso.
- **Pasos:**
  1. Escribir una búsqueda, cerrar el diálogo de Teletransporte y volver a abrirlo — confirmar que
     el buscador vuelve vacío y sigue funcionando.
  2. Colapsar y volver a expandir la sección de Metro varias veces mientras hay texto en el
     buscador.
  3. Con el diálogo abierto y texto en el buscador, rotar el emulador — confirmar que no crashea.
  4. Escribir rápido varios caracteres seguidos — confirmar que no hay lag perceptible.
- **Resultado esperado:** el diálogo y el buscador sobreviven a abrir/cerrar, colapsar/expandir y
  rotación, sin crashes ni comportamiento errático.
- **Resultado real:** el diálogo y el buscador se comportaron según lo esperado en los 4 pasos:
  sin crashes al rotar, el estado se reinició limpiamente al reabrir el diálogo, y no se percibió
  lag al escribir.
- **Estado:** aprobado.
- **Evidencia:** verificación directa en el emulador durante la ejecución.
- **Defecto asociado / decisión:** ninguno.

### A11Y-05 — Accesibilidad: TalkBack en el campo de búsqueda y texto ampliado

- **Criterio cubierto:** criterio de aceptación 1 (el campo debe seguir siendo usable con
  asistencia).
- **Autor / fecha:** José Abel Reyes Castellanos — 1 de octubre de 2026.
- **SHA probado / versión:** `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2` / debug
- **Dispositivo/API:** Medium Phone (AVD), Android 17.0, API 37.1, arm64.
- **Precondiciones y datos:** TalkBack activo; tamaño de fuente del sistema al máximo.
- **Pasos:**
  1. Con TalkBack activo, enfocar el campo de búsqueda de estaciones — confirmar que anuncia su
     etiqueta (ya existente antes de este cambio, no se tocó).
  2. Escribir con el teclado y confirmar que TalkBack no interfiere con la escritura ni con el
     filtrado en vivo.
  3. Con el tamaño de fuente del sistema al máximo, confirmar que el campo y la lista de
     resultados no se rompen ni recortan texto de forma ilegible.
  4. Enfocar con TalkBack un resultado de la lista filtrada y confirmar que anuncia el nombre
     completo de la estación (con su acento) de forma legible.
- **Resultado esperado:** el campo de búsqueda y los resultados siguen siendo operables y
  legibles con TalkBack y fuente ampliada; este cambio no tocó esa parte, pero debe seguir
  funcionando después.
- **Resultado real:** con TalkBack activo el campo de búsqueda y los resultados siguieron siendo
  operables; con el tamaño de fuente del sistema al máximo, el campo y la lista se mantuvieron
  legibles sin recortes.
- **Estado:** aprobado.
- **Evidencia:** verificación directa en el emulador con TalkBack y fuente ampliada activos.
- **Defecto asociado / decisión:** ninguno.

### COMPAT-06 — Compatibilidad/entorno: idioma del dispositivo / distinto nivel de API

- **Criterio cubierto:** el cambio no depende del idioma del sistema (los nombres de estaciones
  son nombres propios, no se traducen).
- **Autor / fecha:** José Abel Reyes Castellanos — 1 de octubre de 2026.
- **SHA probado / versión:** `1b92fed6ec8a53e12e6ed95f90e6c894cb995cb2` / debug
- **Dispositivo/API:** Medium Phone (AVD), Android 17.0, API 37.1, arm64, con el idioma del
  sistema cambiado a English (US).
- **Precondiciones y datos:** idioma del sistema en inglés.
- **Pasos:**
  1. Cambiar el idioma del sistema a inglés.
  2. Repetir la búsqueda `pantitlan` → "Pantitlán" del caso RF-01.
  3. Confirmar que el resultado es el mismo.
- **Resultado esperado:** mismo comportamiento correcto sin importar el idioma del sistema.
- **Resultado real:** con el sistema en inglés, la búsqueda "pantitlan" siguió encontrando
  "Pantitlán" sin diferencias respecto a la prueba en español.
- **Estado:** aprobado.
- **Evidencia:** verificación directa en el emulador con el idioma del sistema en inglés.
- **Defecto asociado / decisión:** ninguno.

## Hallazgos

No se encontraron defectos nuevos durante la ejecución de los 6 casos; los 2 criterios de
aceptación se cumplieron en todos los escenarios probados. Limitaciones documentadas:

- No se auditaron otros buscadores/filtros de texto del proyecto en busca del mismo patrón de
  bug (fuera de alcance declarado); queda como riesgo conocido no cubierto por este PR.
- Las pruebas se corrieron únicamente en el emulador "Medium Phone" (API 37.1); no se probó en un
  dispositivo físico ni en un segundo nivel de API distinto (solo se varió el idioma del sistema
  para COMPAT-06).
- `./gradlew` no funciona de inmediato en una instalación local limpia por el
  `gradle-wrapper.jar` gitignorado (ver nota de entorno); no es un defecto de la app, es
  preexistente en el repositorio y documentado arriba con su solución local.
- El PR Quality Gate de CI no ha corrido en GitHub Actions todavía: GitHub bloquea por seguridad
  la ejecución automática de workflows en PRs desde un fork externo hasta que un mantenedor del
  repo original lo apruebe manualmente ("This workflow is awaiting approval from a maintainer in
  #165"). Bloqueo externo, no relacionado con este cambio; la validación equivalente ya se corrió
  en local con éxito (ver REG-03).

## Cierre del QA

Se recomienda integrar el cambio. Los 2 criterios de aceptación se verificaron con evidencia en
los 6 casos ejecutados (RF-01 a COMPAT-06), más 7 unit tests nuevos (`StationSearchTest`) que
pasan junto con el resto de la suite (`:app:testDebugUnitTest`, `:shared:testAndroidHostTest`) y
una compilación exitosa (`:app:assembleDebug`). El cambio es pequeño, aislado en un archivo nuevo
más dos reemplazos directos de comparador, y no toca persistencia, red ni otra lógica del juego.
Riesgos restantes: no se auditó el resto del proyecto por el mismo patrón de bug, y las pruebas se
limitaron a un solo emulador y nivel de API.
