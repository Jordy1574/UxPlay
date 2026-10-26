# UxPlay para Windows

**Ve la pantalla de tu iPhone o iPad en tu PC, en alta calidad y sin límite de tiempo.**

UxPlay es un **receptor AirPlay de código abierto**: convierte tu computadora en algo parecido a un
Apple TV. Desde el iPhone eliges **Duplicar pantalla** y la imagen y el sonido aparecen en la PC, sin
cables ni apps extra en el teléfono.

- ✅ Gratis y de código abierto (GPL-3.0)
- ✅ Sin anuncios, sin marca de agua y sin límite de tiempo
- ✅ Hasta **4K a 60 fps** (H.265) con decodificación por GPU
- ✅ Imagen y audio sincronizados
- ✅ **Zoom con Ctrl + rueda** sin perder calidad *(agregado en este fork)*
- ✅ **Pantalla completa con F11, doble clic y Esc** *(agregado en este fork)*
- ✅ Funciona aunque tengas VPNs instaladas *(arreglado en este fork)*

> Este repositorio es un fork de [FDH2/UxPlay](https://github.com/FDH2/UxPlay) (versión 1.74),
> adaptado y documentado para **Windows**. Todo el mérito del programa base es de sus autores
> (consulta [Créditos](#créditos)).

---

## Índice

1. [Qué puede hacer](#qué-puede-hacer)
2. [Qué agrega este fork](#qué-agrega-este-fork)
3. [Instalación en Windows](#instalación-en-windows)
4. [Uso](#uso)
5. [Opciones más útiles](#opciones-más-útiles)
6. [Solución de problemas](#solución-de-problemas)
7. [Créditos](#créditos)
8. [Aviso legal y licencia](#aviso-legal-y-licencia)

---

## Qué puede hacer

| Función | Descripción |
|---|---|
| **Duplicar pantalla** | Muestra en la PC todo lo que ves en el iPhone/iPad (AirPlay Mirror). |
| **Solo audio** | Envía música desde el iPhone a la PC en calidad sin pérdida (Apple Lossless). |
| **Video de YouTube** | Admite el envío de videos por AirPlay (HLS), por ahora solo de YouTube. |
| **4K / H.265** | Con la opción `-h265`, el iPhone puede mandar hasta 3840×2160. |
| **Control de acceso** | Puedes pedir un PIN (`-pin`) o una contraseña (`-pw`) para conectarse. |
| **Rotar o voltear** | Gira (`-r`) o voltea (`-f`) la imagen. |

UxPlay usa [GStreamer](https://gstreamer.freedesktop.org/) para mostrar el video y reproducir el
audio, y aprovecha la tarjeta gráfica (Direct3D 12) para decodificar.

**Limitaciones de iOS**, que no dependen de UxPlay:
- El **audio de las llamadas** (WhatsApp, FaceTime, teléfono) no se envía por Duplicar pantalla: solo
  se ve la imagen.
- Algunas apps de streaming bloquean la duplicación de pantalla por protección de contenido.
- No admite audio multisala de AirPlay 2 (para eso existe
  [shairport-sync](https://github.com/mikebrady/shairport-sync)).

---

## Qué agrega este fork

Los cambios son solo para Windows. En Linux y macOS el código se comporta igual que el original.

### 🔍 Zoom con Ctrl + rueda del mouse
- **Ctrl + rueda hacia arriba** acerca y **Ctrl + rueda hacia abajo** aleja.
- El zoom se centra donde está el puntero: lo que tienes debajo del mouse se queda en su lugar.
- Cada paso acerca un 10 %, hasta un máximo de 8×.
- **Sin pérdida de calidad extra:** recorta el video original que manda el iPhone (con `videocrop` de
  GStreamer) en lugar de ampliar una captura de la pantalla. Con `-h265` en 4K se ve todavía más detalle.
- Si cambia la resolución de la transmisión (por ejemplo, al girar el iPhone), el zoom vuelve a 1×.

### 🖥️ Pantalla completa
| Tecla / acción | Resultado |
|---|---|
| **F11** | Activa o desactiva la pantalla completa |
| **Doble clic** sobre el video | Activa o desactiva la pantalla completa |
| **Esc** | Sale de la pantalla completa |

### 🌐 Detección correcta con VPNs
El original anunciaba el receptor en la **primera** red con salida a internet. Si tenías una VPN
(WireGuard/NetBird, Radmin VPN, OpenVPN/TAP…), el anuncio salía por la VPN y **el iPhone nunca veía la
PC**. Ahora se ignoran esos adaptadores y el anuncio sale por tu red local.

Archivos modificados: `lib/mdnsd/mdnsd.c` y `renderers/video_renderer.c`.

---

## Instalación en Windows

**Requisitos:** Windows 10/11 de 64 bits y [MSYS2](https://www.msys2.org/). No hace falta instalar
Bonjour ni iTunes, porque UxPlay 1.74 trae su propio sistema de descubrimiento (mDNS).

### 1. Instala MSYS2
Descárgalo de [msys2.org](https://www.msys2.org/) o, en PowerShell:
```powershell
winget install MSYS2.MSYS2
```

### 2. Instala las dependencias
Abre la terminal **MSYS2 UCRT64** desde el menú Inicio y ejecuta:
```bash
pacman -Syu
pacman -S --needed git mingw-w64-ucrt-x86_64-cmake mingw-w64-ucrt-x86_64-gcc \
  mingw-w64-ucrt-x86_64-ninja mingw-w64-ucrt-x86_64-pkgconf mingw-w64-ucrt-x86_64-libplist \
  mingw-w64-ucrt-x86_64-gstreamer mingw-w64-ucrt-x86_64-gst-plugins-base \
  mingw-w64-ucrt-x86_64-gst-plugins-good mingw-w64-ucrt-x86_64-gst-plugins-bad \
  mingw-w64-ucrt-x86_64-gst-libav mingw-w64-ucrt-x86_64-openssl
```
Si `pacman -Syu` cierra la terminal, ábrela de nuevo y repite el comando.

### 3. Descarga y compila UxPlay
```bash
git clone https://github.com/Jordy1574/UxPlay
cd UxPlay && mkdir build && cd build
cmake .. -G Ninja
ninja
cmake --install . --prefix /ucrt64
```
El programa queda instalado en `C:\msys64\ucrt64\bin\uxplay.exe`.

### 4. Permite UxPlay en la red (una sola vez)
Abre **PowerShell como administrador** y ejecuta:
```powershell
# La red de tu casa debe ser "Privada" (cambia 'Ethernet' por el nombre de tu conexión)
Get-NetConnectionProfile
Set-NetConnectionProfile -InterfaceAlias 'Ethernet' -NetworkCategory Private

# Permite UxPlay en el firewall
New-NetFirewallRule -DisplayName UxPlay -Direction Inbound -Program 'C:\msys64\ucrt64\bin\uxplay.exe' -Action Allow -Profile Private -Protocol Any
```

---

## Uso

### Acceso directo en el escritorio
Crea un archivo `Ver iPhone.bat` en el escritorio con este contenido y ábrelo con doble clic:
```bat
@echo off
title UxPlay - Receptor AirPlay
set PATH=C:\msys64\ucrt64\bin;%PATH%
uxplay.exe -n MiPC -nh -h265 -s 3840x2160@60 -fps 60 -vs d3d12videosink -fs
pause
```
La ventana negra que aparece es UxPlay esperando conexiones: déjala abierta mientras lo usas y ciérrala
para detenerlo.

### En el iPhone o iPad
1. Conéctalo a la **misma red** que la PC.
2. Abre el **Centro de control** (desliza desde la esquina superior derecha).
3. Toca **Duplicar pantalla** y elige **MiPC** (o el nombre que pusiste con `-n`).

### Controles en la PC
| Acción | Resultado |
|---|---|
| Ctrl + rueda ↑ / ↓ | Zoom hacia donde apunta el mouse |
| F11 o doble clic | Pantalla completa / ventana |
| Esc | Salir de pantalla completa |

**Consejo:** el iPhone es más alargado que un monitor 16:9, así que manda la imagen con franjas negras.
Con unos dos pasos de Ctrl + rueda se llena la pantalla. En YouTube, pellizca el video en el iPhone para
que ocupe toda su pantalla.

---

## Opciones más útiles

| Opción | Qué hace |
|---|---|
| `-n nombre` | Nombre con el que aparece la PC en el iPhone |
| `-nh` | No agrega `@nombre-del-equipo` al final del nombre |
| `-h265` | Admite video H.265 (necesario para 4K) |
| `-s 3840x2160@60` | Resolución y frecuencia que se le piden al iPhone (por defecto 1920x1080) |
| `-fps 60` | Cuadros por segundo máximos (por defecto 30) |
| `-vs d3d12videosink` | Usa el reproductor de video Direct3D 12 |
| `-fs` | Abre en pantalla completa |
| `-as 0` | Sin audio, solo imagen |
| `-pin 1234` | Pide un PIN de 4 dígitos para conectarse |
| `-pw clave` | Pide una contraseña para conectarse |
| `-r R` / `-r L` | Gira la imagen 90° a la derecha o a la izquierda |
| `-f H` / `-f V` | Voltea la imagen horizontal o verticalmente |
| `-nc` | No cierra la ventana cuando el iPhone deja de transmitir |
| `-nohold` | Una nueva conexión reemplaza a la actual |
| `-d` | Muestra mensajes de diagnóstico |

Para ver todas las opciones: `uxplay -h`.

---

## Solución de problemas

**El iPhone no encuentra la PC**
- Revisa que la red de la PC esté como **Privada** y que la regla del firewall exista (paso 4).
- El iPhone y la PC deben estar en la misma red. No sirven los datos móviles ni una red de invitados.
- En el iPhone, ve a *Ajustes → Wi-Fi → (i)*: su IP debe empezar igual que la de la PC
  (por ejemplo, `192.168.1.x`).
- Prueba apagar y encender el Wi-Fi del iPhone.

**El video se traba o se ve cortado**
- Usa cable de red en la PC si puedes; por Wi-Fi, mejor en 5 GHz.
- Baja la calidad: quita `-h265` y usa `-s 1920x1080@60`.
- Si tu tarjeta gráfica tiene problemas con la decodificación, prueba `-avdec` (decodificación por software).

**No se escucha el audio**
- Durante una llamada es normal: iOS no envía ese audio.
- Sube el volumen **del iPhone** con los botones laterales mientras transmites.
- Revisa que la salida de audio predeterminada de Windows sea la que usas.

**Windows SmartScreen o el antivirus bloquea `uxplay.exe`**
- Como lo compilas tú mismo, Windows no lo reconoce. Permítelo si lo compilaste desde este repositorio.

---

## Créditos

**UxPlay** es obra de sus autores originales y de la comunidad. Este fork solo agrega las mejoras para
Windows descritas arriba.

- **[FDH2/UxPlay](https://github.com/FDH2/UxPlay):** proyecto original y mantenimiento actual.
- **antimof:** creó UxPlay a partir de RPiPlay, con GStreamer como sistema de audio y video.
- **[RPiPlay](https://github.com/FD-/RPiPlay):** proyecto base de UxPlay.
- **EstebanKubata:** biblioteca FairPlay [PlayFair](https://github.com/EstebanKubata/playfair) (`lib/playfair`).
- **Juho Vähä-Herttua y colaboradores:** servidor de audio AirPlay
  [ShairPlay](https://github.com/juhovh/shairplay), origen de gran parte de `lib/`.
- **dsafa22:** servidor de duplicación AirPlay 2 (AirplayServer); el código de duplicación de `lib/`
  proviene de su trabajo.

La lista completa de autores está en el README del
[proyecto original](https://github.com/FDH2/UxPlay#uxplay-authors).

---

## Aviso legal y licencia

Este proyecto está escrito a partir de información disponible públicamente y tiene fines educativos.
Cada usuario es responsable de cumplir las leyes de su país. Usa una biblioteca GPL de terceros para
FairPlay cuya situación legal no está del todo clara. AirPlay, iPhone e iPad son marcas de Apple Inc.;
este proyecto no está afiliado a Apple.

Se distribuye bajo la licencia **GNU GPL v3**, igual que el original. Consulta [LICENSE](LICENSE).
