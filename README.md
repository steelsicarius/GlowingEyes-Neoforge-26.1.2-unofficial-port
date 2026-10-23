# Glowing Eyes — NeoForge 26.1.2

Unoficial Update to Neoforge 26.1.2 of RaydanOMGr/GlowingEyes

Port de **GlowingEyes** (original de AndreasMelone para NeoForge 21.11.x / MC 1.21.11)
a **NeoForge 26.1.2.114** / **Minecraft 26.1.2** (Java 25).

El original era multiloader (common/fabric/neoforge); este port es **solo NeoForge**
y está basado en el MDK ModDevGradle 2.0.148 (proyecto único).

## Compilar

```bash
JAVA_HOME=/usr/java/jdk-25-oracle-aarch64 ./gradlew build
```

El jar se genera en `build/libs/glowingeyes-2.1.3.jar`.

## Probar

- **Servidor dedicado** (puerto 25599, no interfiere con el servicio del 25565):
  ```bash
  JAVA_HOME=/usr/java/jdk-25-oracle-aarch64 ./gradlew runServer
  ```
  `run/server.properties` ya está configurado con `server-port=25599`,
  `server-ip=127.0.0.1` y `eula=true`.

- **Cliente** (requiere display; en este servidor headless se usa Xvfb):
  ```bash
  xvfb-run -a -s "-screen 0 1280x720x24" JAVA_HOME=/usr/java/jdk-25-oracle-aarch64 ./gradlew runClient
  ```

### Nota sobre el cliente en aarch64

Mojang no publica nativos Linux ARM64 en el manifest de MC 26.1.2. El directorio
`natives-arm64/` (ignorado por git) contiene los nativos arm64 de LWJGL 3.4.1
extraídos de Maven Central, una `libstdc++.so.6` reciente (la del sistema no tiene
`GLIBCXX_3.4.30`) y `libfreetype.so` del sistema. `build.gradle` añade al run de
cliente `-Dorg.lwjgl.librarypath` y `LD_LIBRARY_PATH` apuntando allí. Sin esto el
cliente no arranca en esta máquina; en un PC x86_64 normal no hace falta nada de esto.

## Cambios del port (1.21.11 → 26.1.2)

### Build
- Proyecto único NeoForge con plugin `net.neoforged.moddev` 2.0.148 y
  `neo_version=26.1.2.114` (versiones tomadas del MDK).
- `neoforge.mods.toml` generado desde `src/main/templates/` (sin `modLoader`/`loaderVersion`,
  que ya no se usan); se eliminó el `pack.mcmeta` obsoleto (NeoForge lo gestiona).
- Run de servidor configurado con `--port 25599`.

### Código (APIs de Minecraft 26.x)
- `GuiGraphics` ya no existe: renombrado a `GuiGraphicsExtractor` en todo el código
  (mismo modelo de dibujo: `fill`, `blit`, `blitSprite`, `pose()`, `enableScissor`…).
- Texto: `drawString` → `text`, `drawCenteredString` → `centeredText`.
- Render basado en extract (immediate-mode retirado):
  - `Screen.render` → `Screen.extractRenderState`
  - `Screen.renderBackground` → `Screen.extractBackground`
  - `AbstractWidget.renderWidget` → `extractWidgetRenderState`
  - `AbstractButton/Button.renderContents` → `extractContents`
  - `Button.renderDefaultLabel` → `extractDefaultLabel`
  - `InventoryScreen.renderEntityInInventoryFollowsMouse` → `extractEntityInInventoryFollowsMouse`
  - `GuiGraphics#renderTooltip(font, lines, x, y, positioner, style)` → `GuiGraphicsExtractor#tooltip(...)`
- `LocalPlayer.displayClientMessage(msg, actionBar)` → `LocalPlayer.sendSystemMessage(msg)`
- Pipelines: `RenderPipeline.Builder.withBlend(...)` →
  `.withColorTargetState(new ColorTargetState(...))` (el blend vive ahora en `ColorTargetState`).
- Shaders core actualizados a `#version 330`; el bloque `DynamicTransforms` ya no
  declara `LineWidth`.

### Mixins / AT
- `compatibilityLevel` de los configs subido de `JAVA_17` a `JAVA_21`
  (runtime sponge-mixin 0.17.4 soporta hasta JAVA_25); entradas `refmap` eliminadas
  (no se usan con ModDevGradle).
- Los targets siguen existiendo en 26.x y se aplican en runtime:
  `Minecraft#reloadResourcePacks(Z,GameLoadCookie)`, `AvatarRenderer#extractRenderState`,
  `GuiGraphicsExtractor#innerBlit`, `AvatarRenderState`, `GpuTexture.usage` (AT).
- `META-INF/accesstransformer.cfg` sin cambios (todos los targets siguen presentes).

### Recursos
- Sprites y texturas GUI ya usaban la ruta `textures/gui/sprites/...`: sin cambios.
- `WidgetSprites` ya no tiene constructor de 4 argumentos con nombres, pero el record
  canónico `(enabled, disabled, enabledFocused, disabledFocused)` sigue disponible, así
  que `GuiUtil.createSprites(...)` funciona igual.

## Verificado
- `./gradlew build` verde (jar completo con assets, shaders, mixins y AT).
- Servidor dedicado 26.1.2 en puerto 25599: carga el mod, "Glowing Eyes common setup",
  registros de attachments/paquetes OK, server "Done".
- Cliente 26.1.2 bajo Xvfb: llega al menú principal con "Glowing Eyes client setup",
  keybindings registrados, shaders `glowingeyes:pipeline/*` compilados y recursos
  `mod/glowingeyes` cargados. (El sonido y el narrador se desactivan solos: no hay
  dispositivo de audio en el contenedor.)
- Renderizado de ojos en mundo (capa `GlowingEyesHeadLayer` sobre `AvatarRenderer`) pendiente
  de prueba visual con cliente interactivo; los mixins implicados se aplican sin errores.

## Estructura

```
src/main/java/...           código fusionado common+neoforge del original
src/main/resources/         assets, mixins, AT, servicios
src/main/templates/         neoforge.mods.toml (plantilla con propiedades)
natives-arm64/              (ignorado por git) nativos aarch64 para el run de cliente
run/                        directorio de juego dev (server/client)
```

Licencia: LGPL-3.0 (como el original).
