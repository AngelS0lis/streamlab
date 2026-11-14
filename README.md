# StreamLab — Plataforma de Streaming en Vivo

**StreamLab** es una plataforma ligera y funcional de transmisión de video en vivo al estilo de Twitch, desarrollada como práctica académica con Node.js 22, RTMP, HLS, WebSockets (Socket.io) y SQLite.

## 🚀 Características principales

- **Cuentas y autenticación**: Registro, inicio y cierre de sesión con `bcryptjs` y cookies `HttpOnly` (`sl_token`).
- **Canales personalizables**: Título, categoría, biografía y clave de retransmisión (`stream_key`) regenerable.
- **Ingesta RTMP segura**: Validación de usuario y clave secreta en el evento `prePublish` de `node-media-server` (puerto `1935`), sin exponer la clave en la URL pública del reproductor.
- **Empaquetado HLS y grabación VOD sin recodificar (`-c copy`)**: Conversión en tiempo real con `ffmpeg` a segmentos `.ts` de 2 segundos (`.m3u8`) y grabación automática de cada transmisión en `.mp4` (VOD), más generación de miniaturas en vivo cada 30 segundos.
- **Interacción en tiempo real (Socket.io)**: Chat por canal con colores por usuario, límite anti-spam (700 ms), contador de espectadores en vivo y actualización automática de estado del canal.
- **Explorar y comunidad**: Filtro por 6 categorías (*Videojuegos, Programación, Educación, Música, Charlas, Arte*), sistema de seguidores y pestaña *Siguiendo*.
- **Despliegue automatizado en AWS EC2**: Script `deploy/setup-ec2.sh` para Ubuntu 24.04 (`t3.micro`) con PM2, memoria swap de 1 GB y configuración de puerto 80.

---

## 📁 Estructura del proyecto

```text
streamlab/
├── SPEC.md             # Especificación técnica del proyecto (Anexo A)
├── server.js           # Express + API REST + Socket.io + node-media-server (RTMP/HLS)
├── db.js               # Conexión y esquema SQLite nativo (node:sqlite)
├── package.json        # Dependencias y scripts
├── .env.example        # Plantilla de variables de entorno
├── .gitignore          # Archivos excluidos de Git (node_modules, data, media, .env, *.pem)
├── deploy/
│   └── setup-ec2.sh    # Script de aprovisionamiento y despliegue en Ubuntu 24.04 (EC2)
└── public/
    ├── styles.css      # Estilos visuales (tema oscuro responsivo)
    ├── common.js       # Utilidades compartidas del frontend y barra de navegación
    ├── index.html      # Página Explorar / Categorías / Siguiendo
    ├── channel.html    # Reproductor HLS/VOD + Chat en tiempo real
    ├── login.html      # Inicio de sesión y registro
    └── dashboard.html  # Panel del creador y configuración de OBS
```

---

## 🛠️ Requisitos previos

- **Node.js 22 LTS** (`>= 22.13`, incluye `node:sqlite` integrado)
- **FFmpeg** instalado en el sistema
- **OBS Studio** (o FFmpeg desde terminal para simular transmisiones)
- **Git** y **AWS CLI v2** (para despliegue en AWS EC2)

---

## 💻 Ejecución en local

1. **Instalar dependencias**:
   ```bash
   npm install
   ```

2. **Configurar variables de entorno** (`.env`):
   ```ini
   PORT=3000
   RTMP_PORT=1935
   PUBLIC_HOST=
   FFMPEG_PATH=C:/ruta/a/ffmpeg.exe
   RECORD_VOD=true
   ```

3. **Iniciar el servidor**:
   ```bash
   npm start
   ```
   - Web: `http://localhost:3000`
   - Servidor RTMP (OBS): `rtmp://localhost:1935/live`

4. **Transmitir desde OBS Studio**:
   - **Servicio**: Personalizado
   - **Servidor**: `rtmp://localhost:1935/live`
   - **Clave de retransmisión**: `tu_usuario?key=sk_...` (copiar desde *Mi canal / Panel del creador*)
   - **Salida**: Codificador `x264`, bitrate `2500 Kbps`, intervalo de fotogramas clave `2 s`, resolución `1280x720` a `30 FPS`.

5. **Simular transmisión de prueba con FFmpeg (sin cámara)**:
   ```bash
   ffmpeg -re -f lavfi -i testsrc2=size=1280x720:rate=30 \
     -f lavfi -i sine=frequency=440 \
     -c:v libx264 -preset veryfast -b:v 2500k -g 60 -pix_fmt yuv420p \
     -c:a aac -b:a 128k \
     -f flv "rtmp://localhost:1935/live/TU_USUARIO?key=TU_CLAVE"
   ```

---

## ☁️ Despliegue en AWS EC2 (Ubuntu 24.04)

Dentro de la instancia EC2:

```bash
git clone https://github.com/TU_USUARIO/streamlab.git
cd streamlab
bash deploy/setup-ec2.sh
```
