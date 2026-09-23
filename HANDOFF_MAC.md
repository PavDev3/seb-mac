# Continuación en Mac — SEB dev quit bypass

## Estado del repo
- Fork propio en GitHub: https://github.com/PavDev3/seb-mac (remoto `origin`)
- Repo oficial: https://github.com/SafeExamBrowser/seb-mac (remoto `upstream`)
- Rama `main` de tu fork ya tiene el commit con los cambios (mensaje "initial commit",
  autor pnunfe@gmail.com — quedó auto-commiteado por un hook, contenido correcto).

## Clonar en el Mac
```
git clone https://github.com/PavDev3/seb-mac.git
cd seb-mac
git remote add upstream https://github.com/SafeExamBrowser/seb-mac.git
```

## Qué se implementó
Bypass de quit/force-quit **solo quiero para desarrollo local**, activado por la
variable de entorno `SEB_DEV_BYPASS_QUIT`. Sin esa variable, el comportamiento
es idéntico al original (builds de producción no se ven afectados).

Cambios en `Classes/SEBController.m`:
1. Nuevo helper `SEBDevBypassQuitEnabled()` (después de `@implementation SEBController`,
   ~línea 166) — lee `NSProcessInfo.processInfo.environment[@"SEB_DEV_BYPASS_QUIT"]`.
2. `requestedQuit:` (~línea 9046) — si el bypass está activo, sale directo sin
   comprobar `allowQuit` ni pedir contraseña de salida.
3. `applicationDidFinishLaunching:` (~línea 929) — no deshabilita Force Quit del
   sistema si el bypass está activo.
4. Arranque de Assessment/AAC mode (~línea 4030) — mismo tratamiento.
5. Configuración de kiosko clásico (~línea 7872, la función que se ejecuta en cada
   sesión y sobreescribe las anteriores) — excluye `DisableForceQuit` y
   `DisableSessionTermination` cuando el bypass está activo.

## Pendiente al abrir en Xcode (Mac)
1. Abrir `SafeExamBrowser.xcworkspace` en Xcode.
2. Compilar y verificar que no hay errores (no se pudo compilar-verificar en
   Windows, no hay toolchain de Xcode ahí).
3. Para activar el bypass al probar: Product → Scheme → Edit Scheme → Run →
   Arguments → Environment Variables → agregar `SEB_DEV_BYPASS_QUIT = 1`.
4. Probar: cargar un `.seb` con `allowQuit = NO` y confirmar que Cmd+Q sí cierra
   la app, y que sin la variable de entorno el bloqueo sigue funcionando normal.

## Pendiente sobre Ollama (tema aparte, no bloqueante)
- Se creó un modelo Ollama corregido `qwen3.6-35b-fixed` (bypass de un bug en el
  parser Jinja de Ollama con el chat_template embebido del GGUF original
  `hf.co/HauhauCS/Qwen3.6-35B-A3B-Uncensored-HauhauCS-Aggressive:IQ4_XS`).
- Si en el Mac usas ese modelo local con Claude Code, usa el nombre
  `qwen3.6-35b-fixed` (o `/model qwen3.6-35b-fixed` dentro de una sesión), no el
  nombre original de HuggingFace.
