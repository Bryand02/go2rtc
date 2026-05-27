# go2rtc listo para produccion con 5 camaras

Base de despliegue para `go2rtc` usando Docker Compose, pensada para copiarse al servidor y dejar operativas 5 camaras RTSP.

## Incluye

- `docker-compose.yml` con version fija de imagen
- `.env.example` para parametrizar puertos y version
- `config/go2rtc.example.yaml` con 5 streams predefinidos
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

Primero crea tu archivo local:

```bash
cp config/go2rtc.example.yaml config/go2rtc.yaml
```

Luego edita `config/go2rtc.yaml` y cambia:

- `USUARIO`
- `CLAVE`
- `IP_CAMARA_1` a `IP_CAMARA_5`
- `/stream1` por la ruta RTSP real de cada camara

La IP actual del servidor ya queda definida como:

```yaml
webrtc:
  candidates:
    - 192.168.48.2:8555
```

Ejemplo:

```yaml
cam1:
  - rtsp://admin:123456@192.168.1.101:554/Streaming/Channels/101#timeout=30
```

## 3. Despliegue en Linux

```bash
cp .env.example .env
cp config/go2rtc.example.yaml config/go2rtc.yaml
docker compose pull
docker compose up -d
```

## 4. Puertos a abrir

- `1985/tcp` panel web y API
- `8556/tcp` salida RTSP
- `8557/tcp` y `8557/udp` WebRTC

Si usas `ufw`:

```bash
sudo ufw allow 1985/tcp
sudo ufw allow 8556/tcp
sudo ufw allow 8557/tcp
sudo ufw allow 8557/udp
```

## 5. Verificacion

- Web UI: `http://192.168.48.2:1985/`
- RTSP local: `rtsp://192.168.48.2:8556/cam1`

Diagnostico rapido si no abre desde otro equipo:

```bash
docker compose ps
docker compose logs --tail=50 go2rtc
sudo ss -ltnp | egrep ':1985|:8556|:8557'
sudo ufw status
curl http://127.0.0.1:1985/
```

## Notas

- Esta base usa por defecto `1985`, `8556` y `8557` para evitar conflicto con el `go2rtc` interno de Home Assistant.
- Si el servidor va detras de NAT o proxy, `webrtc.candidates` debe apuntar a la IP o dominio alcanzable por el cliente.
- La version usada queda fijada en `1.9.14`, publicada el 19 de enero de 2026.
- Si una camara tiene stream inestable, puedes cambiar su fuente a `ffmpeg:rtsp://...` mas adelante.
- `config/go2rtc.yaml` queda fuera de Git para que puedas editar camaras en produccion sin romper futuros `git pull`.
