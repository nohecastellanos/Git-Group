# Branch Groups — plugin de IntelliJ

![Licencia: EULA gratuita](https://img.shields.io/badge/licencia-EULA%20gratuita-blue.svg) ![IntelliJ 2022.3+](https://img.shields.io/badge/IntelliJ-2022.3%2B-orange.svg) ![Versión 1.0.0](https://img.shields.io/badge/versi%C3%B3n-1.0.0-green.svg)

Agrupa varios repositorios Git (carpetas en disco) bajo una rama de desarrollo y opera sobre todos a la vez, con las mismas validaciones que hacías con scripts.

## Descripción para JetBrains Marketplace

El texto entre los marcadores se publica como descripción del plugin (el Marketplace exige que esté en inglés). Se inyecta en `plugin.xml` al compilar.

<!-- Plugin description -->
**Branch Groups** manages several Git repositories (plain folders on disk) as a single unit that shares one development branch.

- **Groups**: assign a set of repositories to a branch. When a repository is added, it is fetched and checked out to the group branch, or the branch is created from `develop`. Repositories with uncommitted changes are not added until they are clean.
- **Validate**: see the current branch, expected branch, pending changes and remote state of every repository at a glance.
- **Switch branch**: all repositories are validated first; if any of them is not clean nothing is touched, and a failed checkout is rolled back.
- **Group commit**: review before/after diffs of every changed file across repositories and commit the selected files with one message.
- **Group merge**: predicts conflicts before touching anything and resolves them with the IntelliJ merge tool.
- **Sync, stash and history**: fetch + fast-forward, save/restore uncommitted changes and browse the combined history of the whole group.
- **Maven build**: run Maven across the group's repositories one by one in a defined order, stopping at the first failure. Named execution profiles with per-repository commands, test skipping, -P/-U/offline, JAVA_HOME, settings.xml and MAVEN_OPTS; live console with stop, rerun and resume-from-failure.
- **Export / import settings**: save all groups and settings to an XML file and restore them; an automatic backup outside the IDE lets you recover them after reinstalling.

Uses the command-line `git` executable, so repositories do not need to be open in the IDE.

Requires IntelliJ-based IDEs 2022.3 or later with the bundled Git and Maven plugins; conflict prediction needs Git 2.38+.

Copyright (c) 2026 Nohe Castellanos. Free to use, including in companies, under its End User License Agreement.
<!-- Plugin description end -->

## Requisitos

| | |
|---|---|
| **IDE** | IntelliJ IDEA (Community o Ultimate) u otra IDE de JetBrains 2022.3 o posterior (`sinceBuild = 223`, sin límite superior), incluida la última versión |
| **Plugins del IDE** | Git (`Git4Idea`) y Maven (`org.jetbrains.idea.maven`), incluidos por defecto en IntelliJ IDEA |
| **Git** | Ejecutable `git` instalado; la predicción de conflictos en merge usa `git merge-tree` (Git 2.38+) |
| **Maven** | Solo para "Build Maven": el Maven configurado en *Settings → Build Tools → Maven* |

## Qué hace

| Acción | Comportamiento |
|---|---|
| **Grupos** | Cada grupo tiene nombre, rama asignada y una lista de carpetas. Al agregar un repositorio se hace fetch y, si está limpio, se cambia a la rama del grupo o se crea desde `develop`. Un repositorio con cambios no se incluye hasta que esté OK (commit o stash y "Validar de nuevo"). |
| **Validar** | Por cada repo muestra rama actual, rama esperada y estado: limpio, con cambios (tooltip con los archivos), merge/rebase en curso, o "limpio pero en otra rama". |
| **Cambiar rama** | Fase 1: fetch opcional, valida que *todos* estén limpios y que la rama exista. Si uno falla, no toca ninguno. Fase 2: checkout en todos; si un checkout falla, revierte los que ya cambió. |
| **Merge** | Predice conflictos sin tocar nada (`git merge-tree`, Git 2.38+) y los resuelve con la herramienta de merge de IntelliJ. |
| **Ver cambios** | Visor de diferencias del IDE con los archivos modificados: HEAD (antes) contra el archivo actual (después, editable). F7 / ⇧F7 para navegar. |
| **Commit grupal** | Casillas por archivo, diff antes/después, un mensaje para todos; si un commit falla se deshacen los ya hechos (`reset --soft`). Push opcional. |
| **Sync / stash / historial** | Fetch + fast-forward, stash grupal y restauración, historial combinado. |
| **Build Maven** | Para el grupo seleccionado ejecuta Maven en sus repositorios con `pom.xml`, **uno por uno en el orden que definas**, con el **mismo ejecutor de Maven de IntelliJ** (Maven home, JDK del Runner, VM options, settings.xml, offline y variables de entorno salen de *Settings → Build Tools → Maven*). **Perfiles** (ambientes) por grupo: crear, duplicar, renombrar, eliminar. Cada repositorio tiene su propia configuración (⚙): comando (`clean install -T100 -f pom.xml` por defecto), pruebas (ejecutar / `-DskipTests` / `-Dmaven.test.skip=true`), perfiles `-P`, exigir la rama del grupo y continuar si falla; "Copiar la configuración del seleccionado a los demás" para no repetirla. Si uno falla se detiene la secuencia. La salida se ve en vivo en una pestaña de consola (y en la ventana Run) con **Detener**, **Volver a ejecutar** y **Reanudar desde el que falló**; lanzar otro build detiene el anterior. |
| **Repositorios movidos** | Si la carpeta de un repositorio ya no existe, la tabla lo muestra como "No se encuentra el repositorio" (y el grupo indica cuántos faltan). Doble clic en la fila, o "Editar grupo" → "Reubicar", para seleccionar su nueva carpeta (una por una o buscándolas por nombre dentro de una carpeta); se preparan igual que al agregarlos. |
| **Exportar / importar configuración** | Botones de la barra: guardan los grupos, sus repositorios y los ajustes en un XML, y lo restauran ("Reemplazar todo" o "Combinar"). Si alguna carpeta no existe en el equipo, ofrece ubicarlas antes de importar. |

Los grupos se guardan a nivel de IDE (`options/branchGroups.xml`), así que están disponibles desde cualquier proyecto.

**Respaldo automático**: cada cambio se copia también a `~/.branch-groups/branch-groups-backup.xml` (fuera del IDE).
Si el plugin se desinstala o se pierde la configuración, al volver a abrirlo sin grupos aparece un aviso para restaurar ese respaldo.
Una configuración vacía nunca sobrescribe el respaldo.

## Compilar y probar

Requisitos: JBR/JDK 21 para correr Gradle (configurado en `gradle.properties`) y conexión a internet la primera vez
(descarga IntelliJ IDEA Community 2022.3.3 y, si no lo tienes, el JDK 17 con el que se compila).

Se compila contra **2022.3**, la versión mínima soportada, y para Java 17. Ese mismo instalable funciona en todas
las versiones posteriores. Antes de publicar, `verifyPlugin` lo revisa contra 2022.3 y contra las versiones nuevas.

| Tarea | Qué hace |
|---|---|
| `./gradlew runIde` | Abre un IntelliJ aislado con el plugin (config. **Run IDE with Plugin** en `.run/`) |
| `./gradlew test` | Pruebas con repositorios Git reales (necesita `git` en el PATH) |
| `./gradlew buildPlugin` | Genera el instalable `build/distributions/git-<versión>.zip` |
| `./gradlew verifyPlugin` | Verifica compatibilidad con las IDEs del rango declarado (lo que revisa el Marketplace) |

Instalar a mano: *Settings → Plugins → ⚙ → Install Plugin from Disk…* y elige el zip.

## Publicar en JetBrains Marketplace

Coordenadas: `groupId = com.branchgroups`, `artifactId = git` (`rootProject.name`), id del plugin `com.branchgroups.git` (**no se puede cambiar después de publicar**), versión en `gradle.properties`.

1. **Versión y notas**: sube `version` en `gradle.properties` y agrega su sección en `CHANGELOG.md` (se publica como "What's new").
2. **Certificado de firma** (una sola vez), según https://plugins.jetbrains.com/docs/intellij/plugin-signing.html:
   ```
   openssl genpkey -aes-256-cbc -algorithm RSA -out private_encrypted.pem -pkeyopt rsa_keygen_bits:4096
   openssl rsa -in private_encrypted.pem -out private.pem
   openssl req -key private.pem -new -x509 -days 365 -out chain.crt
   ```
   Guarda esos archivos fuera del repositorio.
3. **Variables de entorno** (nunca en el repo): `CERTIFICATE_CHAIN` (contenido de `chain.crt`), `PRIVATE_KEY` (contenido de `private.pem`), `PRIVATE_KEY_PASSWORD`, y `PUBLISH_TOKEN` (token personal de https://plugins.jetbrains.com/author/me/tokens).
4. **Primera publicación**: `./gradlew signPlugin` y sube a mano `build/distributions/git-<versión>-signed.zip` en https://plugins.jetbrains.com/plugin/add (la primera versión pasa por revisión de JetBrains).
5. **Siguientes versiones**: `./gradlew publishPlugin` (firma, sube y marca la versión en el CHANGELOG).

## Estructura

```
src/main/kotlin/com/branchgroups/
  model/      modelo + validación de nombres de rama
  settings/   persistencia, exportar/importar y respaldo automático
  git/        ejecución del CLI de git, cambios, operaciones de grupo (dos fases)
  maven/      lectura de pom.xml y ejecución de Maven por repositorio
  ui/         tool window, tabla, diálogos, revisión, historial, build Maven, indicador de carga
src/main/resources/META-INF/
  plugin.xml       id, nombre, dependencias y extensiones
  pluginIcon.svg   ícono del Marketplace
src/test/kotlin/   pruebas con repositorios Git temporales
doc/              documentación técnica (arquitectura, funcionalidades, código, build, guía de cambios)
```

## Notas

- Usa el ejecutable de Git configurado en IntelliJ (*Settings → Version Control → Git*). Las credenciales salen de tu configuración de Git; nunca se queda esperando una contraseña (`GIT_TERMINAL_PROMPT=0`).
- Por defecto los archivos *untracked* cuentan como cambios; se puede desactivar en ⚙.
- El remoto asumido es `origin`.

## Autor

**Nohe Castellanos** — <nohe.castellanos@gmail.com>

| Dato | Valor |
|---|---|
| Plugin | Branch Groups |
| Id del plugin | `com.branchgroups.git` |
| groupId / artifactId | `com.branchgroups` / `git` |
| Versión | 1.0.0 |
| Vendor | Nohe Castellanos |

## Licencia

Copyright (c) 2026 Nohe Castellanos. Todos los derechos reservados.

Licencia **propietaria gratuita** (EULA, texto completo en [`LICENSE`](LICENSE)). **El código fuente no es público.**

La licencia **permite**:
- Instalar y usar el plugin gratis, en cualquier cantidad de equipos.
- Usarlo en empresas y con fines comerciales.
- Compartirlo dentro de la misma organización.

La licencia **no permite**:
- Modificarlo, traducirlo ni crear versiones derivadas.
- Descompilarlo ni hacerle ingeniería inversa.
- Venderlo, rentarlo, sublicenciarlo ni redistribuirlo a terceros (se descarga gratis del Marketplace).
- Quitar los avisos de copyright o de licencia.

Además:
- Se entrega **"tal cual"**, sin garantías; el autor no responde por daños o pérdida de datos derivados de su uso.
- No recopila ni envía datos personales ni estadísticas de uso.
- Se rige por las leyes de la República de Guatemala.
- Incluye la librería estándar de Kotlin, que tiene su propia licencia Apache 2.0 (ver [`NOTICE`](NOTICE)).

### En JetBrains Marketplace

Al subir el plugin, en **License** elige la opción de licencia propia (*Other* / *Custom EULA*), no una de código
abierto: así **no se pide enlace al código fuente**. Si el formulario pide la URL de la licencia, publica solo el
texto de `LICENSE` (por ejemplo, en un Gist público o una página propia) y usa ese enlace. Deja vacío el campo
*Source code URL*.

### Si más adelante publicas el código

Puedes cambiar a Apache 2.0 en una versión nueva: reemplaza `LICENSE` por el texto de Apache 2.0, actualiza `NOTICE`
y esta sección, sube el código a un repositorio público y en el Marketplace elige *Apache 2.0* con el enlace al
repositorio. Las versiones ya publicadas con el EULA siguen bajo el EULA.
