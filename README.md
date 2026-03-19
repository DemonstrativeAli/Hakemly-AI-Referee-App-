# Hakemly – AI Psychological Referee Application

**Hakemly** is an AI-powered "psychological referee" application that analyzes couples' voice conversations: it transcribes speech, performs emotion analysis, interprets conversations with LLMs (OpenAI/Anthropic), and provides feedback on who was more constructive, empathetic, or right.

---

## Dual Lane Telemetry

Dual Lane telemetry, WebSocket payload examples, simulator usage, and microphone acceptance steps are documented in `DUAL_LANE_TELEMETRY.md`.

## Performance Trace

The performance trace standard, `/dev/perf/health`, low-quality gating, and the perf harness are documented in `PERFORMANCE_TRACE.md`.

---

## Purpose

The application:

1. **Captures conversations** (microphone or file upload)
2. **Separates speakers** (e.g., who said what)
3. **Analyzes emotions**
4. **Analyzes arguments with ChatGPT/LLM**
5. **Determines who was more right / constructive**
6. **Optionally provides spoken feedback** (TTS)

---

## Folder and File Descriptions

### `models/`

Data schemas and pipeline models.

| File | Description |
|------|-------------|
| `message.py` | Transcript, ChatRequest/Response, StreamingMessageRequest |
| `pipeline.py` | SignalBundle, AgentInput, RawTurn, ConversationContext |
| `__init__.py` | Package init |

---

### `routes/`

API endpoints (external clients communicate through these paths).

| File | Description |
|------|-------------|
| `chat.py` | `/chat/*` – echo, transcripts, history, consent, streaming WebSocket, knowledge, new-architecture analyze |
| `audio.py` | `/audio/live` – real-time diarization WebSocket |
| `dev_panel.py` | `/dev/dual_lane/*` – Dual Lane config, telemetry, simulation, trigger_interrupt |
| `perf.py` | `/dev/perf/health` – performance health summary |
| `optimization.py` | `/optimization/*` – TTS/chat optimization (currently disabled in `main.py`) |

---

### `routers/`

| File | Description |
|------|-------------|
| `chat_website.py` | `/chat_website/*` – website support chatbot (ask, handoff, suggestions, tickets, feedback) |

---

### `services/`

Core AI and audio processing services.

| File | Description |
|------|-------------|
| `llm.py` | LLM integration (OpenAI, Anthropic) |
| `streaming_chat.py` | Real-time message processing, RAG, schema, orchestrator |
| `orchestrator.py` | Schema detection + rule-based agent selection + decision merger |
| `intervention_service.py` | Automatic intervention when communication quality drops |
| `rag_service.py` | RAG retrieval (MongoDB + Elasticsearch) |
| `elasticsearch_service.py` | Elasticsearch vector/full-text search |
| `schema_service.py` | Schema therapy detection and analysis |
| `decision_merger.py` | Merges agent outputs into a single decision |
| `diarize_final_realtime.py` | Real-time diarization (Whisper + pyannote) |
| `emotion_analysis.py` | Emotion analysis |
| `referee_engine.py` | Referee scoring and decisions |
| `text_to_speech.py` | TTS (Hugging Face VITS) |
| `speech_to_text.py` | STT integration |
| `database.py` | MongoDB access |
| `session_state.py` | Session state management |
| `consent_gate.py` | KVKK / consent handling |
| `perf_trace.py` | Performance tracing |
| `text_quality.py` | Text quality gating |

---

### `services/agents/`

Specialist agents invoked by the orchestrator:

| Agent | Description |
|-------|-------------|
| `psychology` | General psychology |
| `mediation` | Mediation |
| `emotion_escalation` | Emotion escalation |
| `legal` | Legal risk |
| `security` | Security / threat detection |
| `schema_therapy` | Schema therapy |

---

### `services/dual_lane/`

Dual Lane (Fast Lane + Truth Lane) pipeline:

| File | Description |
|------|-------------|
| `fast_lane.py` | Fast audio-based triggers (RMS, overlap) |
| `first_referee.py` | Immediate referee interrupts |
| `truth_scheduler.py` | Truth lane job scheduling |
| `truth_worker.py` | Full STT/diarization for truth |
| `incident_assembler.py` | Assembles incidents from truth blocks |
| `input_guard.py` | Blocks legacy chat payloads when truth-first is active |
| `output_gate.py` | Output gating |
| `config_store.py` | Config persistence |
| `dev_control_panel.py` | Dev panel logic |
| `telemetry_hub.py` | Telemetry events |

---

### `utils/`

| File | Description |
|------|-------------|
| `config.py` | Environment / `.env` loading |
| `logging.py` | Logging setup |
| `flags.py` | Feature flags (DUAL_LANE_ENABLED, etc.) |
| `text_formatter.py` | Text formatting helpers |

---

### `main.py`

Application entry point: FastAPI app, router includes, static files, MongoDB/Elasticsearch startup.

---

## Service Flow

```
User sends audio (WebSocket) → [routes/audio.py] /audio/live
    ↓
Real-time Diarization → [diarize_final_realtime.py] (Whisper + pyannote)
    ↓
Dual Lane (optional): Fast Lane (RMS/overlap) → first_referee interrupt
    ↓
Truth Lane: checkpoint blocks → incident_assembler → orchestrator
    ↓
Streaming chat: message → [streaming_chat.py]
    ↓
Preprocessing → Quality gate → RAG retrieval → Schema analysis
    ↓
Orchestrator → Agent selection → Decision merger
    ↓
Intervention (if score low) → [intervention_service.py]
    ↓
LLM response → TTS (optional) → WebSocket / REST response
```

---

## Technologies

| Technology | Purpose |
|------------|---------|
| **FastAPI** | API server |
| **faster-whisper** | Speech-to-text (ASR) |
| **pyannote.audio** | Speaker diarization |
| **HuggingFace Transformers** | Emotion analysis, TTS (VITS) |
| **OpenAI / Anthropic** | LLM API |
| **MongoDB (Motor/PyMongo)** | Main database |
| **Elasticsearch** | Vector/full-text search |
| **sentence-transformers** | RAG embeddings |
| **Docker** | Elasticsearch container |
| **.env** | API keys and configuration |
| **Logging** | Trace and error tracking |

---

## Application Flow

| Step | Description |
|------|-------------|
| 1 | User sends audio via WebSocket (or uses manual text in test UI) |
| 2 | Whisper transcribes speech |
| 3 | pyannote separates speakers (A, B) |
| 4 | Text is analyzed for emotions and schema signals |
| 5 | LLM analyzes via RAG + orchestrator |
| 6 | Agents (psychology, mediation, legal, etc.) provide recommendations |
| 7 | Decision merger produces final output |
| 8 | Optional TTS for spoken feedback |
| 9 | Result shown in UI and/or stored |

---

## Prerequisites

- Python 3.12+
- MongoDB (Atlas or local)
- Elasticsearch (optional; Docker Compose available)
- Hugging Face token (for pyannote embeddings)
- LLM API key (OpenAI or Anthropic)

---

## Running

```bash
# Create venv and install
make venv
make install

# Run server (default: http://127.0.0.1:8000)
make run
# or
invoke run
# or
python -m uvicorn Hakemly.main:app --reload --host 127.0.0.1 --port 8000 --app-dir src
```

Swagger: `http://127.0.0.1:8000/docs`

---

## Testing

```bash
invoke test
# or
python -m Hakemly.tests.test_client
```

`test_client.py` exercises `/chat/echo` and related endpoints. For Dual Lane and performance tests, use `tools/perf_run.py` and the pytest suite in `tests/`.

---

## API Endpoints

### Chat (`/chat`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/chat/consent/status?session_id=` | Consent status |
| POST | `/chat/consent` | Submit consent |
| POST | `/chat/transcripts` | Store transcript |
| POST | `/chat/echo` | LLM response (direct text or transcript reference) |
| GET | `/chat/history?session_id=&limit=` | Chat history |
| POST | `/chat/new-architecture/analyze` | Schema + orchestrator analysis |
| POST | `/chat/knowledge` | Add RAG knowledge |
| GET | `/chat/knowledge/search?query=&limit=` | Search knowledge |
| GET | `/chat/knowledge/stats` | Knowledge stats |
| GET | `/chat/tts?chat_id=` | TTS from chat |
| WS | `/chat/stream` | Streaming chat WebSocket |

### Audio (`/audio`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| WS | `/audio/live` | Real-time diarization WebSocket |

### TTS (root)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/tts?text=` | Text-to-speech streaming |

### Dev Panel (`/dev/dual_lane`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/dev/dual_lane/config` | Dual Lane config snapshot |
| POST | `/dev/dual_lane/config` | Update config |
| GET | `/dev/dual_lane/telemetry` | Telemetry snapshot |
| POST | `/dev/dual_lane/telemetry/simulate` | Simulate telemetry |
| POST | `/dev/dual_lane/trigger_interrupt` | Trigger interrupt |

### Perf (`/dev/perf`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/dev/perf/health` | Performance health summary |

### Website Chatbot (`/chat_website`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/chat_website/ask` | Support chatbot |
| POST | `/chat_website/handoff` | Handoff to live support |
| GET | `/chat_website/suggestions` | Quick suggestions |
| GET | `/chat_website/admin/tickets` | List support tickets |
| PATCH | `/chat_website/admin/tickets/{id}` | Update ticket |
| POST | `/chat_website/feedback` | Submit feedback |

### Optimization (currently disabled)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/optimization/optimized_speak` | Optimized TTS |
| POST | `/optimization/optimized_chat_speak` | Optimized chat + TTS |
| POST | `/optimization/batch_optimized_speak` | Batch TTS |
| GET | `/optimization/performance_stats` | Performance stats |
| GET | `/optimization/health_check` | Health check |
| DELETE | `/optimization/clear_metrics` | Clear metrics |

---

## Summary

**Hakemly** is an AI-driven voice analysis platform that:

- Performs real-time speaker diarization and transcription
- Uses RAG, Elasticsearch, and MongoDB for knowledge and context
- Implements a Dual Lane (Fast + Truth) pipeline with referee interrupts
- Routes through an orchestrator and multiple agents (psychology, mediation, legal, security, schema therapy)
- Provides an intervention service when communication quality drops
- Offers a website support chatbot and dev/telemetry/perf endpoints

Related docs: `DUAL_LANE_TELEMETRY.md`, `PERFORMANCE_TRACE.md`, `RAG_INTEGRATION_COMPLETE.md`, `OPTIMIZATION_GUIDE.md`.
