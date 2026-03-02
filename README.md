# aHoTTS API

API REST de Text-to-Speech (TTS) para **Euskara**, **Español**, **Galego** y **Català**, usando modelos VITS del proyecto [aHoTTS](https://github.com/hitz-zentroa/aHoTTS) de [HiTZ / Aholab](https://huggingface.co/collections/HiTZ/tts) (UPV/EHU).

## Voces disponibles

| Idioma | Código | Voces |
|--------|--------|-------|
| Euskara (Basque) | `eu` | antton, maider |
| Galego (Galician) | `gl` | brais, celtia, iago, icia, paulo, sabela |
| Català (Catalan) | `ca` | bet, eli, eva, jan, mar, ona, pau, pep, pol |
| Español (Spanish) | `es` | laura, alejandro |

## Inicio rápido

### 1. Construir y levantar con Docker Compose

```bash
docker compose up --build -d
```

La API estará disponible en `http://localhost:8000`.

### 2. Documentación interactiva

- **Swagger UI:** http://localhost:8000/docs
- **ReDoc:** http://localhost:8000/redoc

## Endpoints

### `GET /health`
Estado del servicio.

```bash
curl http://localhost:8000/health
```

### `GET /voices`
Lista todas las voces disponibles por idioma y su estado de descarga.

```bash
curl http://localhost:8000/voices
```

### `GET /voices/{language}`
Lista las voces disponibles para un idioma específico.

```bash
curl http://localhost:8000/voices/eu
```

### `POST /synthesize`
Sintetiza texto a audio WAV.

```bash
curl -X POST http://localhost:8000/synthesize \
  -H "Content-Type: application/json" \
  -d '{"text": "Kaixo, zer moduz zaude?", "language": "eu", "voice": "antton"}' \
  --output audio.wav
```

### `GET /synthesize`
Sintetiza por query parameters (útil para pruebas rápidas).

```bash
curl "http://localhost:8000/synthesize?text=Hola+mundo&language=es&voice=laura" \
  --output audio.wav
```

### `POST /download/{language}/{voice}`
Pre-descarga un modelo de voz desde HuggingFace (la primera síntesis también lo descarga automáticamente).

```bash
curl -X POST http://localhost:8000/download/eu/antton
```

## Ejemplos por idioma

```bash
# Euskara
curl -X POST http://localhost:8000/synthesize \
  -H "Content-Type: application/json" \
  -d '{"text": "Antton naiz, zer moduz zaude.", "language": "eu", "voice": "antton"}' \
  -o euskara.wav

# Español
curl -X POST http://localhost:8000/synthesize \
  -H "Content-Type: application/json" \
  -d '{"text": "Soy Laura, ¿qué tal estás?", "language": "es", "voice": "laura"}' \
  -o espanol.wav

# Galego
curl -X POST http://localhost:8000/synthesize \
  -H "Content-Type: application/json" \
  -d '{"text": "Son Brais, como estás.", "language": "gl", "voice": "brais"}' \
  -o galego.wav

# Català
curl -X POST http://localhost:8000/synthesize \
  -H "Content-Type: application/json" \
  -d '{"text": "Sóc Ona, com estàs.", "language": "ca", "voice": "ona"}' \
  -o catala.wav
```

## Configuración

Variables de entorno:

| Variable | Descripción | Valor por defecto |
|----------|-------------|-------------------|
| `MAX_TEXT_LENGTH` | Longitud máxima del texto de entrada | `5000` |
| `AHOTTS_BASE_PATH` | Ruta base del proyecto aHoTTS | `/app/aHoTTS` |

## Arquitectura

```
aHoTTS_API/
├── app/
│   ├── __init__.py
│   ├── config.py        # Configuración y constantes
│   ├── engine.py        # Motor TTS (descarga modelos + ejecuta binario)
│   ├── main.py          # Aplicación FastAPI con endpoints
│   └── models.py        # Modelos Pydantic (request/response)
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── README.md
```

## Notas técnicas

- Los modelos VITS se descargan automáticamente desde HuggingFace en la primera petición de síntesis para cada voz.
- Los modelos se persisten en un volumen Docker (`ahotts-voices`) para no re-descargarlos entre reinicios.
- La caché de HuggingFace también se persiste en un volumen (`ahotts-hf-cache`).
- El binario `tts` incluido en el repo de hitz-zentroa/aHoTTS es un ejecutable Linux x86_64.
- La API usa FastAPI con uvicorn y soporta peticiones asíncronas concurrentes.

## Licencia

- **Código API:** MIT
- **aHoTTS:** [Apache License 2.0](https://github.com/hitz-zentroa/aHoTTS/blob/main/LICENSE)
- **Modelos de voz:** consultar las licencias individuales en [HuggingFace](https://huggingface.co/collections/HiTZ/tts)

## Créditos

- [HiTZ - Basque Center for Language Technology](https://www.hitz.eus/)
- [Aholab Signal Processing Laboratory (UPV/EHU)](https://aholab.ehu.eus/)
- Modelos VITS basados en [Kim et al. (2021)](https://arxiv.org/abs/2106.06103)
