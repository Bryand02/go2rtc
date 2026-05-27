# go2rtc listo para produccion con 5 camaras

Base de despliegue para `go2rtc` usando Docker Compose, pensada para copiarse al servidor y dejar operativas 5 camaras RTSP.

## Incluye

- `docker-compose.yml` con version fija de imagen
- `.env.example` para parametrizar puertos y version
- `config/go2rtc.yaml` con 5 streams predefinidos
- logs rotados del contenedor para evitar crecimiento infinito

## 1. Preparar variables

Crea un archivo `.env` a partir de `.env.example`.

```env
GO2RTC_VERSION=1.9.14
GO2RTC_API_PORT=1985
GO2RTC_RTSP_PORT=8556
GO2RTC_WEBRTC_PORT=8557
```

## 2. Configurar el servidor

Edita `config/go2rtc.yaml` y cambia:

- `SERVER_IP_O_DNS` por la IP o dominio del servidor
- `USUARIO`
- `CLAVE`
- `IP_CAMARA_1` a `IP_CAMARA_5`
- `/stream1` por la ruta RTSP real de cada camara

Ejemplo:

```yaml
cam1:
  - rtsp://admin:123456@192.168.1.101:554/Streaming/Channels/101#timeout=30
```

## 3. Despliegue en Linux

```bash
cp .env.example .env
docker compose pull
docker compose up -d
```

## 4. Puertos a abrir

- `1985/tcp` panel web y API
- `8556/tcp` salida RTSP
- `8557/tcp` y `8557/udp` WebRTC

## 5. Verificacion

- Web UI: `http://IP_DEL_SERVIDOR:1985/`
- RTSP local: `rtsp://IP_DEL_SERVIDOR:8556/cam1`

## Notas

- Esta base usa por defecto `1985`, `8556` y `8557` para evitar conflicto con el `go2rtc` interno de Home Assistant.
- Si el servidor va detras de NAT o proxy, `webrtc.candidates` debe apuntar a la IP o dominio alcanzable por el cliente.
- La version usada queda fijada en `1.9.14`, publicada el 19 de enero de 2026.
- Si una camara tiene stream inestable, puedes cambiar su fuente a `ffmpeg:rtsp://...` mas adelante.
