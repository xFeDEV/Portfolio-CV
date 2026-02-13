## 🎬 AutoClipMaker

Sistema para **detectar momentos “clippeables” en VODs de Kick.com** analizando la actividad del chat (hype), y opcionalmente **descargar el clip en MP4**.

- **Frontend**: Vue 3 + Vite (dev) y Nginx (prod)
- **Backend**: FastAPI + SQLAlchemy + ffmpeg
- **BD**: PostgreSQL (pensado para Supabase)
- **Data Lake** (cache): JSON crudo del chat en disco (`back-autoclipmaker/storage/chats/*.json`)

---

## 📌 Qué problema resuelve

Cuando un streamer tiene un momento viral, el chat “explota”. AutoClipMaker:

- Descarga el chat de un VOD de Kick.
- Calcula una señal de **hype** en el tiempo.
- Detecta picos y propone **clips** (rangos de tiempo) ordenados por “virabilidad”.
- Guarda resultados (BD + cache) para que futuros análisis del mismo VOD sean instantáneos.

---

## 🧠 Arquitectura (visión global)

```text
Usuario (Browser)
  │
  ▼
Frontend (Vue)  ───────────────┐
  │                            │
  │ 1) Kick APIs               │ 2) Backend APIs
  │ (video + chat)             │ (analyze, streams, clips, video)
  ▼                            ▼
Kick.com API                Backend (FastAPI)
                               │
                               ├─ PostgreSQL/Supabase (streams, clips)
                               ├─ Data Lake (storage/chats/*.json)
                               ├─ Debug logs (debug_logs/*.json)
                               └─ ffmpeg (convert TS → MP4)
```

---

## 🔄 Flujo end-to-end (lo que pasa realmente)

### 1) Nuevo análisis (VOD → clips)

1. **Usuario pega** un link de Kick (VOD).
2. Frontend extrae `vod_id` y hace **cache check**:
   - `GET /api/streams/check/{vod_id}`
   - Si hay **CACHE HIT**, el backend devuelve `{ analysis }` (hype_timeline + clips) sin descargar nada.
3. Si hay **CACHE MISS**, el frontend:
   - `GET https://kick.com/api/v1/video/{vod_uuid}` → metadata (channel_id, start_time, duration, title)
   - `GET https://kick.com/api/v2/channels/{channel_id}/messages?start_time=...` en segmentos/paralelo
4. El frontend arma el payload `ChatData` y lo manda a:
   - `POST /analyze` (endpoint legacy montado “sin /api”, se mantiene por compatibilidad)
5. El backend:
   - Normaliza timestamps del chat a un “epoch” fijo para trabajar con tiempos relativos.
   - Calcula score por mensaje (risas + emotes whitelist + combo).
   - Genera una señal de hype con **ventana deslizante**.
   - Detecta picos con estrategia **Highest Peak First**.
   - Ajusta inicio/fin del clip con un **método por derivada** (punto de inflexión), para recortar contexto innecesario.
6. Persistencia/caching:
   - Escribe `debug_logs/01..03_*.json` (input original, timestamps normalizados, resultados).
   - Escribe Data Lake: `storage/chats/{vod_id}.json`.
   - Upsert de Stream + inserción de Clips en la BD.
7. Devuelve `potential_clips` + `hype_timeline` al frontend.

### 2) Biblioteca (reusar análisis)

- Listado de streams analizados:
  - `GET /api/streams`
  - **Optimizado** para evitar N+1: el backend usa un `JOIN + GROUP BY` para traer `clips_count` en una sola query.
- Re-análisis sin re-descargar chat:
  - `POST /api/streams/reanalyze/{vod_id}`
  - Lee el JSON del Data Lake, re-ejecuta el algoritmo actual y reemplaza clips/timeline.
- Eliminación:
  - `DELETE /api/streams/{stream_id}`
  - Borra stream + clips + JSON del Data Lake.

### 3) Descarga del clip en MP4 (TS → MP4)

El frontend descarga y une segmentos HLS del VOD en un `.ts`, y lo sube al backend:

- `POST /api/video/convert/ts-to-mp4?filename=...` (multipart/form-data)

El backend convierte con ffmpeg:

- Primero intenta **stream copy** (rápido, sin recodificar).
- Si falla, usa **re-encoding** (H.264 + AAC) para compatibilidad.

Notas:
- El frontend muestra **progreso real** de descarga del MP4 leyendo el stream (`ReadableStream`).
- Nginx aumenta `client_max_body_size` y timeouts para soportar archivos grandes.

---

## 🧩 Componentes clave del backend

### Estructura (clean architecture)

```text
back-autoclipmaker/
  app/
    main.py                  # FastAPI app + routers
    api/
      v1/
        endpoints/           # analysis, streams, clips, video
    services/
      analysis_service.py    # HypeAnalyzer (lógica pura)
    crud/                    # acceso a datos (streams/clips)
    models/                  # SQLAlchemy models
    schemas/                 # Pydantic (requests/responses)
    core/
      config.py              # Settings (env)
      database.py            # engine + session (pool + SSL)
  storage/chats/             # Data Lake (cache)
  debug_logs/                # JSONs de debug
  data/                      # artefactos (ej: gráfica)
```

### Concurrencia (detalle importante)

Varios endpoints son **sync** (no `async def`) a propósito: FastAPI los ejecuta en un **thread pool**, evitando bloquear el event loop cuando el trabajo es CPU/IO pesado (análisis, DB, etc.).

En producción se configura uvicorn con **múltiples workers** (cuando `DEBUG=false`).

---

## 🧠 Algoritmo de detección (resumen técnico)

- **Normalización temporal**: convierte timestamps absolutos de Kick a un timeline relativo al VOD (epoch fijo + offset `segundoInicialStream`).
- **Scoring por mensaje**:
  - Base 1.0
  - +3 si detecta risa (`j/a` predominante)
  - +4 por emote en whitelist (`[emote:id:NOMBRE]`)
  - *Combo multiplier* si hay risa + emote
- **Señal de hype**: suma de scores en ventana deslizante.
- **Picos**: umbral tipo `mean + k*std` y selección Highest Peak First.
- **Inicio/fin óptimo**: usa derivada de la señal para encontrar puntos donde la pendiente cambia significativamente (inicio de reacción / fin del decay).

Los parámetros viven en `AnalysisConfig` (`app/services/analysis_service.py`).

---

## 📡 API (lo esencial)

Base:
- **Legacy (sin prefijo)**: `POST /analyze` (usado por el frontend)
- **Versionado v1 (prefijo /api)**: todo lo demás bajo `/api/*`

### Endpoints principales

- **Health**
  - `GET /health`

- **Análisis**
  - `POST /analyze` (legacy)
  - `POST /api/analyze` (mismo router, bajo /api)

- **Streams (biblioteca/cache)**
  - `GET /api/streams`
  - `GET /api/streams/check/{vod_id}`
  - `POST /api/streams/reanalyze/{vod_id}`
  - `GET /api/streams/pending`
  - `DELETE /api/streams/{stream_id}`

- **Clips**
  - `GET /api/clips/stream/{stream_id}`
  - `PUT /api/clips/{clip_id}` (status: pending/approved/rejected)
  - `DELETE /api/clips/{clip_id}`

- **Video**
  - `POST /api/video/convert/ts-to-mp4`
  - `GET /api/video/health`
  - `DELETE /api/video/cleanup`

Documentación interactiva:
- Swagger: `GET /docs`
- Redoc: `GET /redoc`

---

## ⚙️ Configuración (.env)

Archivo: `.env` (usa `.env.example` como plantilla)

Variables importantes:
- **DATABASE_URL** (requerida en producción): conexión PostgreSQL/Supabase.
- **DEBUG**: `true/false`.
- **DB_POOL_SIZE / DB_MAX_OVERFLOW**: tamaño del pool para concurrencia.
- **CHAT_STORAGE_DIR / DEBUG_LOGS_DIR / DATA_DIR**: rutas internas.

Nota importante:
- El backend está **orientado a PostgreSQL/Supabase**. Si quieres usar SQLite en local, revisa la validación de `DATABASE_URL` en `back-autoclipmaker/app/core/config.py`.

---

## 🐳 Ejecutar con Docker

### Opción A) Dev simple (Nginx + backend)

```bash
docker compose up -d --build
# Frontend: http://localhost:3000
# Backend:  http://localhost:8000 (docs en /docs)
```

### Opción B) Dev con hot-reload (Vite + uvicorn --reload)

```bash
docker compose -f docker-compose.dev.yml up -d --build
# Frontend (Vite): http://localhost:5173
# Backend:         http://localhost:8000
```

### Opción C) Producción (Traefik + SSL)

Lee `DEPLOY.md`.

Resumen:

```bash
./deploy.sh setup
./deploy.sh start
```

---

## 🧪 Desarrollo sin Docker

Backend:

```bash
cd back-autoclipmaker
pip install -r requirements.txt
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Frontend:

```bash
cd kick-chat-downloader
pnpm install
pnpm dev
```

---

## 🗂️ Dónde se guarda todo

- **BD**: PostgreSQL/Supabase (`streams`, `clips`).
- **Data Lake (cache)**: `back-autoclipmaker/storage/chats/{vod_id}.json`.
- **Debug**: `back-autoclipmaker/debug_logs/01..03_*.json`.
- **Artefactos**: `back-autoclipmaker/data/` (ej: `hype_analysis.png`).
- **Temporales de video**: `/tmp/video_processing` dentro del contenedor del backend.

---

## 🛠️ Troubleshooting rápido

- **502 / timeouts al convertir a MP4**:
  - Asegúrate de que el backend tenga `ffmpeg` (en Docker ya se instala).
  - Revisa que Nginx/Traefik tengan timeouts altos (ya están aumentados).

- **DB no conecta (Supabase)**:
  - Revisa `DATABASE_URL` y que sea `postgresql://...`.
  - En prod/Supabase se fuerza `sslmode=require`.

- **CACHE HIT no funciona**:
  - Verifica que el stream tenga `is_chat_downloaded=true` y `hype_timeline_json`.
  - Confirma que existe el JSON en `storage/chats/`.

---

## 📝 Nota sobre estado del repo

- Este README refleja **también cambios locales no commiteados** (concurrencia, pool DB, optimización de queries, timeouts de Nginx y flujo TS→MP4 con progreso).
- `kick-chat-downloader/README.md` es el template de Vite/Vue (no es la documentación real del sistema).

---

## 📄 Licencia

MIT.