# UxPlay para Windows: zoom, pantalla completa y VPN

Fork de [FDH2/UxPlay](https://github.com/FDH2/UxPlay), el receptor AirPlay de código abierto.
Con él puedes duplicar la pantalla de un iPhone o iPad en una PC con Windows, sin anuncios,
sin marca de agua y sin límite de tiempo.

Este fork se basa en UxPlay 1.74 y agrega mejoras solo para **Windows**. En Linux y macOS el
código se comporta igual que el original.

## Cambios respecto al original

### 1. Zoom con Ctrl + rueda del mouse
- Con el puntero sobre el video, **Ctrl + rueda hacia arriba** acerca y **Ctrl + rueda hacia abajo** aleja.
- El zoom se centra en el punto donde está el puntero: lo que tienes debajo del mouse se queda en su lugar.
- Cada paso acerca un 10 %, hasta un máximo de 8×.
- El zoom **recorta el video decodificado** (con un elemento `videocrop` de GStreamer) en lugar de
  ampliar la imagen de la pantalla, así que no pierde calidad extra: se ven los píxeles reales que
  manda el iPhone.
- Si cambia la resolución de la transmisión (por ejemplo, al girar el iPhone), el zoom vuelve a 1×.

### 2. Pantalla completa con F11, doble clic y Esc
| Acción | Resultado |
|---|---|
| **F11** | Activa o desactiva la pantalla completa |
| **Doble clic** (botón izquierdo) sobre el video | Activa o desactiva la pantalla completa |
| **Esc** | Sale de la pantalla completa |

Funciona con `d3d12videosink` y con cualquier reproductor de video que tenga la propiedad `fullscreen`.

### 3. El anuncio AirPlay ya no se va por una VPN
El mDNS integrado de UxPlay elegía la **primera** red que tuviera puerta de enlace. Si la PC tiene
una VPN (WireGuard/NetBird, Radmin VPN, OpenVPN/TAP…), el anuncio salía por la VPN y el
iPhone nunca veía el receptor. Ahora se ignoran los adaptadores cuyo nombre contiene
`WireGuard`, `VPN` o `TAP-`, y el anuncio sale por tu red local.

Archivos modificados:
- `lib/mdnsd/mdnsd.c`: filtro de adaptadores VPN.
- `renderers/video_renderer.c`: zoom y pantalla completa.

## Compilar en Windows

Requisitos: Windows 10/11 de 64 bits y [MSYS2](https://www.msys2.org/).
No hace falta el Bonjour SDK, porque UxPlay 1.74 trae su propio mDNS.

1. Instala MSYS2 (por ejemplo, con `winget install MSYS2.MSYS2`) y abre la terminal **MSYS2 UCRT64**.
2. Actualiza e instala las dependencias:
   ```bash
   pacman -Syu
   pacman -S --needed git mingw-w64-ucrt-x86_64-cmake mingw-w64-ucrt-x86_64-gcc \
     mingw-w64-ucrt-x86_64-ninja mingw-w64-ucrt-x86_64-pkgconf mingw-w64-ucrt-x86_64-libplist \
     mingw-w64-ucrt-x86_64-gstreamer mingw-w64-ucrt-x86_64-gst-plugins-base \
     mingw-w64-ucrt-x86_64-gst-plugins-good mingw-w64-ucrt-x86_64-gst-plugins-bad \
     mingw-w64-ucrt-x86_64-gst-libav mingw-w64-ucrt-x86_64-openssl
   ```
3. Descarga este fork en la rama con los cambios y compílalo:
   ```bash
   git clone -b windows-zoom-fullscreen https://github.com/Jordy1574/UxPlay
   cd UxPlay && mkdir build && cd build
   cmake .. -G Ninja
   ninja
   cmake --install . --prefix /ucrt64
   ```
   El ejecutable queda en `C:\msys64\ucrt64\bin\uxplay.exe`.

## Uso

Puedes crear un archivo `.bat` en el escritorio para abrirlo con doble clic:

```bat
@echo off
set PATH=C:\msys64\ucrt64\bin;%PATH%
uxplay.exe -n MiPC -nh -h265 -s 3840x2160@60 -fps 60 -vs d3d12videosink -fs
pause
```

| Opción | Para qué sirve |
|---|---|
| `-n MiPC -nh` | Nombre con el que aparece la PC en el iPhone |
| `-h265 -s 3840x2160@60` | Pide video 4K en H.265: más detalle, sobre todo al hacer zoom |
| `-fps 60` | Hasta 60 cuadros por segundo |
| `-vs d3d12videosink -fs` | Usa el reproductor Direct3D 12 y abre en pantalla completa |

En el iPhone: **Centro de control → Duplicar pantalla →** elige el nombre de tu PC.

Si el video se traba en 4K, quita `-h265` y usa `-s 1920x1080@60`.

## Solución de problemas

- **El iPhone no encuentra la PC.**
  - La red de la PC debe estar como **Privada**, no Pública.
  - `uxplay.exe` tiene que estar permitido en el firewall. Desde una PowerShell como administrador:
    ```powershell
    New-NetFirewallRule -DisplayName UxPlay -Direction Inbound -Program 'C:\msys64\ucrt64\bin\uxplay.exe' -Action Allow -Profile Private -Protocol Any
    ```
  - El iPhone y la PC deben estar en la misma red, no en una red de invitados.
- **No se escucha el audio de una llamada (WhatsApp, FaceTime, etc.).** Es normal: iOS no manda
  el audio de las llamadas por Duplicar pantalla. La música y los videos sí se escuchan.
- **Bordes negros alrededor del video.** Los manda el iPhone, porque su pantalla tiene otra
  proporción que la del monitor. Ajusta con Ctrl + rueda: unos dos pasos suelen llenar una pantalla 16:9.

## Licencia

GPL-3.0, igual que el proyecto original. Consulta [LICENSE](LICENSE).
Todo el mérito de UxPlay es de sus autores; este fork solo agrega las mejoras descritas arriba.
