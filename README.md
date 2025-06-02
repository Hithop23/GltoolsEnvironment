Aquí tienes el texto transformado en formato Markdown bien organizado:

```markdown
# PCGLTools

Spoofea y personaliza la información de tu GPU en Linux usando `LD_PRELOAD` y un lanzador GUI.

PCGLTools es una herramienta que te permite interceptar y modificar la información que las aplicaciones obtienen sobre tu tarjeta gráfica (nombre, fabricante, VRAM, versiones de API, extensiones) en sistemas Linux. Utiliza la técnica de inyección de librerías (`LD_PRELOAD`) y proporciona una interfaz gráfica para facilitar la gestión de perfiles de configuración por aplicación.

Además del spoofing de información, el proyecto sienta las bases para un overlay en el juego que muestre datos en tiempo real y la configuración de spoofing activa.

## Características

- **Spoofing de APIs Gráficas**: Intercepta llamadas a funciones de OpenGL, GLX, EGL y Vulkan para modificar la información reportada sobre la GPU.
- **Perfiles por Aplicación**: Configura diferentes ajustes de spoofing (nombre, VRAM, versiones, extensiones) para cada juego o aplicación usando archivos de configuración YAML.
- **Lanzador GUI**: Una aplicación gráfica que simplifica el proceso de seleccionar perfiles y ejecutar programas con la inyección de la librería activada.
- **Modo "Real GPU"**: Opción en el lanzador para ejecutar aplicaciones sin inyección, mostrando la información real de tu hardware.
- **Overlay en el Juego (En Desarrollo)**: Estructura para dibujar un HUD dentro de las aplicaciones que muestre FPS, temperatura e información de spoofing.

## Estructura del Proyecto

```
├── CMakeLists.txt             # Archivo principal para compilar con CMake
├── README.md                  # Este archivo
├── configs                    # Directorio para archivos de configuración
│   ├── global_config.yaml     # Configuración global por defecto
│   └── profiles               # Perfiles específicos por aplicación
│       ├── csgo.yaml          # Ejemplo de perfil para CS:GO
│       ├── steam_app_12345.yaml # Ejemplo de perfil para una app de Steam
│       └── valorant.yaml      # Ejemplo de perfil para Valorant
├── include                    # Archivos de encabezado (.h)
│   ├── config_loader.h        # Declaraciones para la carga de configuración
│   ├── hooks.h                # Declaraciones para hooks generales
│   ├── injector.h             # Encabezado principal de la librería inyectada
│   ├── overlay.h              # Declaraciones para la lógica del overlay
│   ├── shared_state.h         # Declaraciones para variables globales compartidas
│   ├── spoof_opengl.h         # Declaraciones para hooks de OpenGL/GLX/EGL
│   └── spoof_vulkan.h         # Declaraciones para hooks de Vulkan
├── src                        # Archivos fuente (.cpp)
│   ├── config_loader.cpp      # Implementación de la carga de configuración
│   ├── hooks.cpp              # Implementación de hooks generales
│   ├── launcher.cpp           # Código del lanzador GUI
│   ├── overlay.cpp            # Implementación de la lógica del overlay
│   ├── spoof_opengl.cpp       # Implementación de hooks de OpenGL/GLX/EGL
│   ├── spoof_vulkan.cpp       # Implementación de hooks de Vulkan
│   └── utils.cpp              # Funciones de utilidad (FPS, temp, strings)
└── install_dependencies.sh    # Script para instalar dependencias del sistema
```

## Instalación de Dependencias

Para compilar PCGLTools, necesitas instalar algunas librerías de desarrollo del sistema. En sistemas basados en Debian/Ubuntu (como Linux Mint), puedes usar el script proporcionado:

```bash
chmod +x install_dependencies.sh
sudo ./install_dependencies.sh
```

**Nota**: Este script instala las dependencias del sistema. También necesitarás obtener el código fuente de Dear ImGui manualmente y añadirlo a tu proyecto (se sugiere en `thirdparty/imgui/`) para la funcionalidad del overlay.

## Compilación

El proyecto utiliza CMake para la compilación. Sigue estos pasos desde la raíz del proyecto:

1. Crea un directorio de build:
   ```bash
   mkdir build
   ```

2. Navega al directorio de build:
   ```bash
   cd build
   ```

3. Configura el proyecto con CMake. Asegúrate de que CMake pueda encontrar todas las dependencias (los mensajes de salida de CMake te indicarán si falta algo):
   ```bash
   cmake ..
   ```

   Si CMake encuentra problemas con las dependencias (especialmente si no usaste el script de instalación o si faltan backends de ImGui), deberás resolverlos antes de continuar.

4. Compila el proyecto:
   ```bash
   make
   ```

Si la compilación es exitosa, encontrarás los archivos compilados (`pcgltools_injector.so` y `pcgltools-launcher`) en el directorio `build/`.

## Configuración

La configuración se gestiona a través de archivos YAML en el directorio `configs/`.

- `configs/global_config.yaml`: Contiene la configuración por defecto para todos los ajustes de spoofing y opciones generales.
- `configs/profiles/`: Este directorio contiene archivos de perfil específicos para aplicaciones individuales. El nombre del archivo (ej: `csgo.yaml`, `valorant.yaml`) idealmente debe coincidir con el nombre del ejecutable de la aplicación. Los valores definidos en un archivo de perfil sobrescriben los valores correspondientes en `global_config.yaml`.

Puedes editar estos archivos YAML con un editor de texto para personalizar el spoofing.

## Uso del Lanzador GUI

El lanzador GUI (`pcgltools-launcher`) es la forma recomendada de usar PCGLTools.

1. Primero, asegúrate de haber compilado el proyecto (`make`).
2. Ejecuta el lanzador desde el directorio `build/`:
   ```bash
   ./pcgltools-launcher
   ```

En la ventana del lanzador:

- **Selecciona un Perfil (.cfg/.yaml)**: Usa el selector de archivos o la lista de perfiles disponibles para elegir el archivo de configuración que deseas usar para la sesión de juego (ya sea `global_config.yaml` o un perfil específico de `configs/profiles/`).
- **Selecciona el Ejecutable**: Usa el selector de archivos para elegir el archivo ejecutable del juego o programa que deseas lanzar.
- **Botón "✨ LAUNCH WITH SPOOF ✨"**: Haz clic en este botón para lanzar la aplicación seleccionada con la librería inyectada (`pcgltools_injector.so`) y el perfil de configuración elegido activado.
- **Botón "🔵 LAUNCH REAL GPU 🔵"**: Haz clic en este botón para lanzar la aplicación seleccionada sin inyectar la librería. Esto es útil para comparar el comportamiento o si encuentras problemas con el spoofing.

El lanzador creará un nuevo proceso para el juego, configurará las variables de entorno necesarias (`LD_PRELOAD`, `PCGLTOOLS_PROFILE`) y luego ejecutará el juego. La ventana del lanzador permanecerá abierta.

## Uso Avanzado (LD_PRELOAD Manual)

Si prefieres no usar el lanzador GUI, puedes inyectar la librería manualmente usando la variable de entorno `LD_PRELOAD` en la terminal.

1. Asegúrate de haber compilado la librería (`make`).
2. Configura `LD_PRELOAD` con la ruta absoluta a `build/pcgltools_injector.so`.
3. Opcionalmente, configura `PCGLTOOLS_PROFILE` con la ruta absoluta al archivo de perfil `.yaml` que deseas usar. Si no configuras `PCGLTOOLS_PROFILE`, la librería intentará cargar un perfil por nombre de ejecutable o `global_config.yaml`.
4. Ejecuta el comando de la aplicación.

**Ejemplo** (usando el perfil de CS:GO y asumiendo que estás en la raíz del proyecto):
```bash
LD_PRELOAD=$(pwd)/build/pcgltools_injector.so PCGLTOOLS_PROFILE=$(pwd)/configs/profiles/csgo.yaml /ruta/al/ejecutable/de/csgo
```

## Overlay en el Juego

La funcionalidad de overlay está en desarrollo. Una vez implementada, si el modo debug está activado en el perfil cargado, o si hay una opción específica en la configuración para el overlay, deberías ver una pequeña ventana dentro del juego mostrando información como FPS, temperatura y los valores de spoofing aplicados.

## Advertencias

- **Anticheat**: El uso de `LD_PRELOAD` y la modificación del comportamiento de las APIs gráficas pueden ser detectados por sistemas anticheat intrusivos en juegos online. Estos sistemas están diseñados para identificar cualquier código externo que intente interactuar o modificar el proceso del juego. `LD_PRELOAD` es una técnica común utilizada por herramientas de cheating, lo que la convierte en un objetivo principal para la detección. Los sistemas anticheat pueden buscar la presencia de librerías inesperadas cargadas, verificar la integridad del código de las funciones de la API gráfica, o detectar respuestas inusuales a las consultas de hardware. Si PCGLTools es detectado, las consecuencias pueden variar desde una expulsión temporal de la partida hasta la suspensión permanente de tu cuenta de juego. La efectividad de los sistemas anticheat varía significativamente entre juegos. Úsalo bajo tu propia responsabilidad.

- **Estabilidad**: Modificar las respuestas de las APIs gráficas puede causar inestabilidad, fallos o artefactos visuales en algunas aplicaciones. Los juegos y programas a menudo esperan respuestas específicas y un comportamiento predecible del driver gráfico real. Al spoofear esta información, aunque la aplicación pueda iniciar, podría encontrar inconsistencias o errores lógicos que lleven a cierres inesperados, congelamientos, o problemas de renderizado (texturas corruptas, geometría incorrecta, efectos faltantes).

- **Compatibilidad**: La compatibilidad con diferentes GPUs, drivers y versiones de APIs puede variar. Algunas aplicaciones realizan verificaciones de hardware muy estrictas o dependen de características específicas del driver que la información spoofed podría no reflejar con precisión. Esto podría resultar en que el juego no se inicie en absoluto, que ciertas características gráficas no estén disponibles, o que el rendimiento sea peor de lo esperado a pesar de reportar hardware de gama alta.

## Contribuciones

¡Siéntete libre de contribuir, reportar problemas o sugerir mejoras!
```
