# 🌌 ORION | Optical Recognition & Intelligent Obstacle Navigation

<div align="center">

[![Python](https://img.shields.io/badge/Python-3.10%20%7C%203.11%20%7C%203.12%20%7C%203.13-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)](https://pytorch.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![ESP32-S3](https://img.shields.io/badge/Hardware-Seeed%20XIAO%20ESP32S3-E7352C?style=for-the-badge&logo=espressif&logoColor=white)](https://www.seeedstudio.com/)
[![Android](https://img.shields.io/badge/Android-Kotlin%20%7C%20Compose%20%7C%20TFLite-3DDC84?style=for-the-badge&logo=android&logoColor=white)](https://developer.android.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)

**Sistema Ciberfísico de Visión Artificial en Tiempo Real para Asistencia y Navegación de Personas con Discapacidad Visual basado en Estimación de Profundidad Monocular (MiDaS), Streaming IoT a 30 FPS y Tolerancia a Fallos mediante Conmutación en Caliente (Hot-Swap).**

</div>

---

<div align="center">
  <img src="images/portada.jpeg" alt="ORION - Sistema de Navegación y Detección de Obstáculos" width="100%" style="border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.15);">
</div>

---

## 📑 Tabla de Contenidos

1. [Descripción General](#-descripción-general)
2. [Demostración en Video](#-demostración-en-video)
3. [Arquitectura del Sistema y Árbol de Proyecto](#-arquitectura-del-sistema-y-árbol-de-proyecto)
4. [Evolución de Versiones (v1 a v6)](#-evolución-de-versiones-v1-a-v6)
5. [Requisitos Previos del Sistema](#-requisitos-previos-del-sistema)
6. [Guía de Instalación Paso a Paso](#-guía-de-instalación-paso-a-paso)
7. [Instrucciones de Ejecución](#-instrucciones-de-ejecución)
   - [Modo 1 Clic (Recomendado)](#modo-1-clic-lanzador-automático)
   - [Modo Consola CLI y Argumentos](#modo-consola-cli-y-parámetros-avanzados)
8. [Variables de Entorno (.env)](#-variables-de-entorno-env)
9. [Protocolo de Red, Endpoints y Telemetría](#-protocolo-de-red-endpoints-y-telemetría)
10. [Batería de Pruebas y Diagnóstico](#-batería-de-pruebas-y-diagnóstico)
11. [Higiene de Git y Seguridad](#-higiene-de-git-y-seguridad)
12. [Créditos, Mentoría y Licencia](#-créditos-mentoría-y-licencia)

---

## 🎯 Descripción General

**ORION** (*Optical Recognition & Intelligent Obstacle Navigation*) es una solución de asistencia sensorial diseñada para proporcionar autonomía y seguridad a personas con discapacidad visual en desplazamientos interiores y exteriores.

El sistema resuelve el reto de la percepción espacial en tiempo real sin requerir sensores LiDAR costosos ni cámaras estereoscópicas pesadas. Utilizando un único flujo de video monocular de bajo consumo transmitido por microcontroladores **Seeed Studio XIAO ESP32S3 Sense**, ORION infiere la distancia métrica relativa de cada punto del entorno mediante redes neuronales profundas (**MiDaS v2.1 Small / DPT**).

### Capacidades Destacadas:
* **Estimación de Profundidad Monocular Continua:** Mapeo de calor inverso (*Magma / Inferno*) y cuantificación de peligro por zonas (*Izquierda, Centro, Derecha*).
* **Streaming Inalámbrico Optimizado (22-30 FPS):** Firmware especializado con reloj DMA desacoplado y descarte de frames por saturación TCP en redes Wi-Fi institucionales.
* **Tolerancia a Fallos y Rollback en Caliente (*Hot-Swap*):** Ante cualquier pérdida de paquetes de radio o apagado del microcontrolador, conmuta automáticamente a la cámara local en < 1 segundo sin reiniciar el servicio ni interrumpir la navegación.
* **Auto-Descubrimiento Cero Configuración:** Detección de dispositivos mediante Faro UDP en puerto 9999, verificación cripto-visual JPEG (SOI `0xFFD8`) y caché de IPs (`known_xiao_ips.json`).
* **Multiplataforma Integral:** Funciona como servidor web interactivo en **PC / Mac** (con aceleración Apple Silicon Metal MPS) y como aplicación móvil autónoma en **Android** (Kotlin, TFLite GPU OpenCL/Vulkan y modo bolsillo con pantalla apagada).

---

## 🎥 Demostración en Video

Demostración del mapeador de proyección y detección espacial en funcionamiento:

[![ORION - Demostración del Sistema](https://img.youtube.com/vi/UGBBGHXeREA/maxresdefault.jpg)](https://youtu.be/UGBBGHXeREA)

---

## 🏗️ Arquitectura del Sistema y Árbol de Proyecto

```text
ORION/
├── iniciar_proyecto.bat          # 🚀 Lanzador 1 Clic para Windows (auto-venv, deps y run)
├── iniciar_proyecto.sh           # 🚀 Lanzador 1 Clic para macOS y Linux (chmod +x)
├── main.py                       # Lanzador universal multi-versión (v5 Web por defecto)
├── main_web_xiao.py              # Wrapper de retrocompatibilidad para servicios de inicio macOS (.sh)
├── pyproject.toml                # Especificación estándar de empaquetado (PEP 517/621)
├── requirements.txt              # Dependencias de producción certificadas (PC / Linux / Mac)
├── requirements-mac.txt          # Dependencias optimizadas para Apple Silicon (Metal MPS)
├── .env.example                  # Plantilla maestra documentada de variables de entorno
├── .gitignore                    # Reglas estrictas de higiene (excluye .env, venv, tflite, logs)
│
├── versions/                     # Implementaciones modulares por arquitectura y entorno
│   ├── v1_desktop_initial/       # Versión 1: Prototipo de escritorio síncrono OpenCV directo
│   ├── v2_desktop_optimized/     # Versión 2: Escritorio multihilo con telemetría HUD y alertas
│   ├── v3_web_local/             # Versión 3: Servidor Web FastAPI con cámara web local
│   ├── v4_web_esp32/             # Versión 4: Servidor Web adaptado a ESP32-CAM AI-Thinker
│   ├── v5_web_xiao_esp32s3/      # Versión 5: [INSIGNIA PC/MAC] Web XIAO ESP32S3 + Hot-Swap
│   │   ├── main.py               # Servidor FastAPI, WebSocket 25Hz y conmutación de cámaras
│   │   ├── start.py              # Runner auxiliar con detección automática de dispositivo
│   │   └── firmware/             # Código fuente Arduino / ESP-IDF para XIAO ESP32S3
│   │       └── XIAO_Camera_Orion/# Sketch con Wi-Fi HT20, streaming DMA y baliza UDP 9999
│   └── v6_mobile_android/        # Versión 6: [MÓVIL AUTÓNOMO] App Nativa Android + Hotspot
│       ├── android/              # Proyecto Android Studio (Kotlin, Jetpack Compose, TFLite GPU)
│       ├── firmware/             # Firmware XIAO optimizado para Hotspot móvil
│       └── tools/                # Scripts Python de paridad y exportación del modelo TFLite
│
├── docs/                         # Documentación técnica de nivel de ingeniería
│   ├── ARCHITECTURE.md           # Flujo de datos, formalismo matemático de MiDaS y latencias
│   ├── VERSIONS.md               # Análisis comparativo y matriz de evolución de v1 a v6
│   ├── HARDWARE_XIAO_ESP32S3.md  # Conexión de hardware, antena externa y pinout
│   ├── HARDWARE_ESP32_CAM.md     # Guía de flasheo FTDI para ESP32-CAM AI-Thinker
│   ├── MACOS_SERVICE.md          # Configuración de arranque como demonio en macOS
│   └── API_AND_TELEMETRY.md      # Especificación formal de API REST y WebSockets
│
├── templates/
│   └── index.html                # Interfaz web HUD compartida (Canvas, WebSockets y Manómetros)
├── utils/
│   └── xiao_discovery.py         # Motor de escaneo de red, faros UDP y verificación JPEG SOI
├── tests/                        # Scripts de diagnóstico de red, puertos y estrés de cámara
└── images/                       # Recursos gráficos y diagramas ilustrativos
```

---

## 🗂️ Evolución de Versiones (v1 a v6)

| Versión | Nombre Clave | Plataforma | Fuente de Video | Inferencia IA | Alertas | Estado |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: |
| **v1** | `v1_desktop_initial` | Desktop | Cámara Local USB | CPU (Síncrono) | Consola | Archivo histórico |
| **v2** | `v2_desktop_optimized` | Desktop | Cámara Local USB | CPU/CUDA (Multihilo) | HUD + Audio local | Archivo histórico |
| **v3** | `v3_web_local` | Servidor Web | Cámara Local USB | CPU/CUDA/MPS | Web UI 25 Hz | Estable |
| **v4** | `v4_web_esp32` | Servidor Web | ESP32-CAM AI-Thinker | CPU/CUDA/MPS | Web UI + Buzzer | Estable |
| **v5** | `v5_web_xiao_esp32s3` | Servidor Web | **XIAO ESP32S3 Sense** + Rollback Cámara PC/Mac | **CUDA / MPS / CPU** | **Web + Buzzer IoT + Audio** | **Producción (PC/Mac)** |
| **v6** | `v6_mobile_android` | Móvil Nativo | **XIAO ESP32S3 Sense** (Hotspot Móvil) | **GPU Móvil (FP16) / CPU XNNPACK** | **Háptica + Audio Binaural + Voz TTS** | **Producción (Móvil)** |

---

## 💻 Requisitos Previos del Sistema

### Software Requerido:
* **Python:** Versión `3.10`, `3.11`, `3.12` o `3.13` (64 bits).
* **Git:** Para clonar y gestionar el repositorio.
* **Navegador Web Moderno:** Google Chrome, Edge, Safari o Firefox con soporte de WebSockets y Canvas HTML5.
* **Para Desarrollo Móvil (Solo versión 6):** Android Studio Ladybug (2024.2+) o superior con Android SDK 34 y NDK.
* **Para Firmware IoT (Opcional si ya está flasheado):** Arduino IDE 2.x con el paquete de tarjetas `esp32` por Espressif versión `>= 2.0.11`.

### Hardware Recomendado:
* **Computadora Central (v1 a v5):**
  - **Windows / Linux:** Procesador con 4+ núcleos, 8 GB RAM. Tarjeta gráfica NVIDIA (GTX 1060 o superior) recomendada para inferencia ultrarrápida (CUDA).
  - **macOS:** MacBook Air / Pro con Apple Silicon (M1, M2, M3 o M4) con aceleración nativa por Metal Performance Shaders (MPS).
* **Microcontrolador IoT:** Seeed Studio XIAO ESP32S3 Sense (con módulo de cámara OV2640 y antena externa conectada).
* **Dispositivo Móvil (v6):** Teléfono con Android 10 (API 29) o superior, con soporte de aceleración gráfica OpenGL ES 3.1 / Vulkan.

---

## 📦 Guía de Instalación Paso a Paso

### 1. Clonar el Repositorio
```bash
git clone https://github.com/tu-centro/OrionBelt.git
cd OrionBelt
```

### 2. Crear y Activar el Entorno Virtual
Se recomienda aislar las dependencias en un entorno virtual (`.venv`):

* **En Windows (PowerShell / CMD):**
  ```powershell
  python -m venv .venv
  .\.venv\Scripts\activate
  ```

* **En macOS / Linux:**
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate
  ```

### 3. Instalar las Dependencias Certificadas

* **En Windows y distribuciones Linux:**
  ```bash
  pip install --upgrade pip
  pip install -r requirements.txt
  ```

* **En macOS (Apple Silicon M1/M2/M3/M4):**
  ```bash
  pip install --upgrade pip
  pip install -r requirements-mac.txt
  ```

### 4. Configurar el Archivo de Entorno (`.env`)
Copia la plantilla oficial y personaliza tus parámetros:
```bash
# En Windows:
copy .env.example .env

# En macOS / Linux:
cp .env.example .env
```

---

## 🚀 Instrucciones de Ejecución

### Modo 1 Clic (Lanzador Automático)

El proyecto incluye scripts interactivos que validan el entorno virtual, instalan dependencias faltantes y arrancan el sistema sin intervención manual:

* **En Windows:** Haz doble clic en [`iniciar_proyecto.bat`](iniciar_proyecto.bat) o ejecútalo desde terminal:
  ```cmd
  iniciar_proyecto.bat
  ```
* **En macOS / Linux:** Otorga permisos de ejecución y lanza [`iniciar_proyecto.sh`](iniciar_proyecto.sh):
  ```bash
  chmod +x iniciar_proyecto.sh
  ./iniciar_proyecto.sh
  ```

---

### Modo Consola CLI y Parámetros Avanzados

Ejecuta el lanzador central universal [`main.py`](main.py):

```bash
# 1. Arrancar la versión insignia v5 con detección automática de hardware
python main.py

# 2. Forzar ejecución en Apple Silicon (Metal MPS) o NVIDIA (CUDA)
python main.py --device mps --port 8000
python main.py --device cuda --port 8000

# 3. Especificar una IP estática para la cámara XIAO (omite escaneo de red)
python main.py --xiao-ip 172.16.121.142

# 4. Ajustar el umbral de detección de obstáculos (0.0 a 1.0)
python main.py --near-threshold 0.65

# 5. Listar todas las versiones disponibles en el sistema
python main.py --list

# 6. Ejecutar una versión histórica específica
python main.py --version 2    # Ejecuta v2_desktop_optimized
python main.py --version 3    # Ejecuta v3_web_local
```

Una vez iniciado el servidor, abre tu navegador web en:
👉 **[http://localhost:8000](http://localhost:8000)** (o la IP local de tu equipo en la red LAN).

---

## ⚙️ Variables de Entorno (.env)

| Variable | Tipo / Valores | Por Defecto | Descripción |
| :--- | :--- | :---: | :--- |
| `ORION_VERSION` | `1`, `2`, `3`, `4`, `5`, `6` | `5` | Versión que arranca por defecto al ejecutar `main.py`. |
| `ORION_HOST` | IPv4 (`str`) | `0.0.0.0` | Dirección de enlace (`0.0.0.0` para acceso LAN, `127.0.0.1` solo local). |
| `ORION_PORT` | Entero (`int`) | `8000` | Puerto TCP del servidor web y WebSockets. |
| `XIAO_IP` | IPv4 o vacío | `172.16.121.142` | IP del microcontrolador XIAO (vacío activa autodescubrimiento). |
| `XIAO_STREAM_PORT` | Entero (`int`) | `81` | Puerto TCP del stream de video MJPEG del microcontrolador. |
| `XIAO_CTRL_PORT` | Entero (`int`) | `80` | Puerto HTTP para control remoto de buzzer y telemetría. |
| `FALLBACK_SRC` | Índice (`int`) o URL | `0` | Índice de cámara local para el rollback autónomo ante desconexión. |
| `DEVICE` | `auto`, `cuda`, `mps`, `cpu` | `auto` | Dispositivo de cómputo para la inferencia de la red neuronal MiDaS. |
| `MODEL_TYPE` | `MiDaS_small`, `DPT_Hybrid`, `DPT_Large` | `MiDaS_small` | Variante del modelo MiDaS (Small optimizado para tiempo real). |
| `PROC_WIDTH` | Entero (`int`) | `320` | Ancho de imagen de entrada para preprocesamiento. |
| `TEMPORAL_ALPHA` | Float (`0.0` a `1.0`) | `0.65` | Coeficiente del filtro temporal EMA para eliminar parpadeo. |
| `NEAR_THRESHOLD` | Float (`0.0` a `1.0`) | `0.60` | Umbral relativo de proximidad central para detonar alarma `STOP`. |
| `BUZZER_DUTY` | Entero (`0` a `255`) | `200` | Potencia PWM del buzzer piezoeléctrico en el cinturón. |
| `BUZZER_DURATION` | Entero (`ms`) | `150` | Duración del pitido acústico en milisegundos. |
| `ENABLE_LOCAL_SOUND` | Boolean (`true`/`false`) | `true` | Activa alertas sonoras emitidas por los altavoces de la PC/Mac. |

---

## 🌐 Protocolo de Red, Endpoints y Telemetría

El servidor FastAPI expone los siguientes endpoints para el frontend, dispositivos de telemetría y asistentes externos:

| Endpoint | Método | Protocolo | Descripción |
| :--- | :---: | :---: | :--- |
| `/` | `GET` | HTTP / HTML5 | Panel de control web interactivo (HUD en vivo, manómetro y telemetría). |
| `/stream` | `GET` | MJPEG / HTTP | Flujo de video procesado con el mapa de calor Magma y HUD superpuesto. |
| `/ws` | — | WebSocket | Transmisión bidireccional a 25 Hz de métricas de proximidad, zonas y FPS. |
| `/api/telemetry` | `GET` | JSON | Instantánea del estado actual: FPS, latencia, zonas y estado de cámaras. |
| `/api/switch_source` | `POST` | JSON | Forzar conmutación manual de cámara (`XIAO_ESP32S3` $\leftrightarrow$ `LOCAL_FALLBACK`). |
| `/buzzer?on=1&duty=200` | `GET` | HTTP | Comando de prueba directo hacia el microcontrolador XIAO ESP32S3. |

---

## 🧪 Batería de Pruebas y Diagnóstico

El proyecto incorpora herramientas para validar la conectividad del microcontrolador y la estabilidad del flujo de video:

```bash
# 1. Ejecutar las pruebas automatizadas del proyecto
pytest

# 2. Diagnóstico integral de interfaces de red y puertos locales
python tests/diagnose_network.py

# 3. Escaneo y localización automática del ESP32-CAM / XIAO en la subred
python tests/find_esp32.py

# 4. Prueba de estrés de conexión TCP y streaming HTTP con el microcontrolador
python tests/test_esp32.py
```

---


## 👥 Créditos, Mentoría y Licencia

* **Proyecto:** ORION (*Optical Recognition & Intelligent Obstacle Navigation*)
* **Entidad / Centro:** Prácticas Profesionales de Ingeniería de Software & Inteligencia Artificial - AudacIA
* **Licencia:** Este proyecto se distribuye bajo la licencia **MIT**. Consulta el archivo `LICENSE` para más detalles.

<div align="center">
  <sub>Construido con dedicación para transformar la autonomía y calidad de vida de las personas con discapacidad visual.</sub>
</div>
