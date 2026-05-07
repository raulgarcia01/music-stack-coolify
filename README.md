# 🎵 Stack de Música — Guía de Uso

## Servicios incluidos

| Servicio | Puerto | Función |
|---|---|---|
| **Navidrome** | 4533 | Servidor de música |
| **yt-dlp Web UI** | 3033 | Descargador con interfaz web |
| **Picard (web)** | 8086 | Tagging de metadata (artista, álbum, carátula) |
| **Essentia** | — | Tagging automático de géneros por IA (se ejecuta manualmente) |

---

## 🚀 Primera vez — setup y arranque

```bash
# 1. Copiar y editar variables de entorno
cp .env.example .env
nano .env   # cambia YTDLP_JWT_SECRET, YTDLP_USER y YTDLP_PASS

# 2. Construir imagen de Essentia (tarda unos minutos)
docker compose build essentia

# 3. Levantar todo el stack
docker compose up -d
```

---

## ⬇️ Flujo completo: descarga → metadata → géneros

### Paso 1 — Descargar desde YouTube (Web UI)

1. Abre `http://TU_IP:3033` en el navegador
2. Inicia sesión con el usuario/password que definiste en `.env`
3. Pega la URL de YouTube (canción o playlist) y presiona descargar
4. Monitorea el progreso en tiempo real desde la misma UI

El archivo cae en:
```
/home/botadmin/Music/<NombreDelCanal>/<TítuloDeLaCanción>.mp3
```

### Paso 2 — Corregir metadata con Picard

1. Abre `http://TU_IP:8086` en el navegador
2. Arrastra la carpeta recién descargada al panel izquierdo
3. Click en **Cluster** → luego **Lookup**
4. Verifica que los matches sean correctos (verde = buena coincidencia)
5. Click en **Save** — Picard escribe artista, álbum y carátula correctos

### Paso 3 — Tagging de géneros con Essentia

```bash
# ⚠️ Siempre hacer dry-run primero para ver qué va a cambiar
docker compose run --rm essentia --dry-run

# Si se ve bien, correr en modo real
docker compose run --rm essentia
```

Essentia analiza el audio y escribe tags como:
```
GENRE = Hip-Hop - Gangsta; Hip-Hop - East Coast Hip Hop
MOOD  = Energetic; Dark
```

### Paso 4 — Navidrome re-escanea automáticamente

Escanea tu librería cada hora (`ND_SCANSCHEDULE=1h`).  
Para forzar un escaneo inmediato: **Navidrome → Settings → Rescan Library**.

---

## 📻 Agregar radios de internet

Se ejecuta directamente en el host (no como contenedor):

```bash
# Instalar dependencia
pip3 install requests

# Descargar el script
curl -o /usr/local/bin/navidrome-radio \
  https://raw.githubusercontent.com/WB2024/Add-Navidrome-Radios/main/navidrome-radio.py
chmod +x /usr/local/bin/navidrome-radio

# Ejecutar
navidrome-radio
```

Cuando te pida la ruta de la base de datos, usa:
```
./data/navidrome.db
```

---

## 📁 Estructura de carpetas del proyecto

```
.
├── docker-compose.yml
├── .env                     # ⚠️ Nunca subir a git — contiene passwords
├── .env.example             # Plantilla pública
├── data/                    # Base de datos de Navidrome
├── picard/
│   └── config/              # Configuración de Picard
├── essentia/
│   ├── Dockerfile           # Build local de Essentia
│   └── models/              # Modelos ML descargados (~87MB, persistentes)
└── ytdlp/
    ├── config/
    │   └── config.yml       # Config de yt-dlp-web-ui (formato, output path)
    └── cookies/             # Cookies de YouTube (si las necesitas)
```

---

## 🔑 Agregar Spotify / Last.fm después

Cuando tengas las keys, descomenta en `docker-compose.yml`:

```yaml
- 'ND_SPOTIFY_ID=${ND_SPOTIFY_ID:-}'
- 'ND_SPOTIFY_SECRET=${ND_SPOTIFY_SECRET:-}'
- 'ND_LASTFM_APIKEY=${ND_LASTFM_APIKEY:-}'
- 'ND_LASTFM_SECRET=${ND_LASTFM_SECRET:-}'
```

Y agrégalas a un archivo `.env` en la misma carpeta:

```env
ND_SPOTIFY_ID=tu_client_id
ND_SPOTIFY_SECRET=tu_client_secret
ND_LASTFM_APIKEY=tu_api_key
ND_LASTFM_SECRET=tu_secret
```
