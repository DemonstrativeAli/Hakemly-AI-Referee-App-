# 🧠 Hakemly – Psikolojik Hakem Uygulaması

**Hakemly**, çiftlerin sesli konuşmalarını analiz ederek, metne çeviren, duygu analizi yapan, konuşmaları LLM (ChatGPT) ile yorumlayan ve sonunda kimin daha yapıcı/empatik/haklı olduğuna dair geri bildirim sunan bir **"psikolojik hakem" yapay zeka uygulamasıdır**.

---

## 🎯 Amaç

Bu uygulama:

1. **Konuşmaları alır** (mikrofonla veya dosya yükleme ile)
2. **Her kişiyi ayırır** (örneğin: Ali ne dedi, Ayşe ne dedi)
3. **Duygu analizini yapar**
4. **ChatGPT ile argümanları analiz eder**
5. **Kimin haklı olduğunu belirler**
6. (Opsiyonel) **Sesli geri bildirim verir**

---

## 🧱 Proje Yapısı

```bash
.
├── Dockerfile                          # Docker container image configuration for the application
├── Makefile                            # Make commands for project management (run, test, clean, install)
├── OPTIMIZATION_GUIDE.md               # Optimization guide and performance improvement documentation
├── RAG_INTEGRATION_COMPLETE.md         # RAG (Retrieval-Augmented Generation) integration documentation
├── README.md                           # Main project documentation, setup and usage guide
├── docker-compose.yml                  # Docker Compose configuration for all services (MongoDB, API)
├── pyproject.toml                      # Python project configuration and package metadata
├── requirements.txt                    # Python dependencies (FastAPI, Whisper, transformers, etc.)
├── tasks.py                            # Invoke task runner - defines run, test commands
├── scripts/                            # Helper scripts directory
│   └── populate_knowledge.py           # Script to populate RAG knowledge base with psychological information
├── src/                                # Main source code directory
│   └── Hakemly/                        # Main application package
│       ├── main.py                     # FastAPI application entry point, registers routes, manages MongoDB connection
│       ├── models/                     # Pydantic models and data schemas
│       │   ├── __init__.py             # Module initialization
│       │   ├── message.py              # API models (ChatRequest, ChatResponse, TranscriptCreate, etc.)
│       │   └── order.py                 # Referee decisions and order models (currently empty)
│       ├── routes/                     # FastAPI route endpoints
│       │   ├── __init__.py             # Main router that combines all routers
│       │   ├── audio.py                # /audio/live WebSocket endpoint - receives live audio stream, processes STT
│       │   ├── chat.py                 # /chat endpoints - LLM responses, transcript storage, RAG knowledge management
│       │   └── optimization.py         # /optimization endpoints - performance optimization and batch processing
│       ├── services/                   # Business logic services
│       │   ├── __init__.py             # Services module initialization
│       │   ├── database.py             # MongoDB connection and CRUD operations (transcripts, chats, knowledge)
│       │   ├── emotion_analysis.py     # Emotion analysis service (currently only function definition)
│       │   ├── llm.py                  # OpenAI GPT API communication, generates RAG-enhanced responses
│       │   ├── memory.py               # Conversation history and memory management (currently empty)
│       │   ├── message_engine.py       # Message processing engine (currently empty)
│       │   ├── optimization.py         # Batch processing, performance metrics, TTS optimization
│       │   ├── product_matcher.py      # Product matching service (currently empty)
│       │   ├── rag_service.py         # RAG service - retrieves relevant knowledge from database, generates embeddings
│       │   ├── referee_engine.py       # Referee decision engine - analyzes who is right (currently only function definition)
│       │   ├── speech_to_text.py      # Whisper-based STT service - converts audio files to text
│       │   └── text_to_speech.py      # Hugging Face TTS service - converts text to speech with caching
│       ├── static/                     # Static frontend files
│       │   └── index.html              # Main web interface - live audio recording via WebSocket, chat interface
│       ├── tests/                      # Test files directory
│       │   └── test_client.py          # Simple test client for API endpoint testing
│       └── utils/                      # Helper functions and configuration
│           ├── __init__.py             # Utils module initialization
│           ├── config.py               # Reads environment variables (.env), manages settings
│           ├── logging.py             # Logging configuration and logger factory
│           └── text_formatter.py      # Formats LLM responses for TTS, extracts objective evaluation section
└── hakemly.egg-info/                   # Python package metadata (created after installation)
```

## 📁 Klasör ve Dosya Açıklamaları

### `models/`
Veritabanı ve veri şemaları tutulur.

| Dosya | Görevi |
|-------|--------|
| `message.py` | Kim ne dedi, hangi konuşma kime ait |
| `order.py` | Hakem kararı, analiz sonucu gibi yapılar |
| `__init__.py` | Modül başlangıcı |

---

### `routes/`
API endpoint'leri (yani dış dünya bu yolları kullanarak uygulamayla iletişim kurar)

| Dosya | Görevi |
|-------|--------|
| `chat.py` | /chat endpointi üzerinden ChatGPT’ye metin gönderimi |
| `audio.py` | /analyze-audio endpointi üzerinden ses yükleme ve analiz işlemi 🆕 |

---

### `services/`
Uygulamanın yapay zekâ ve ses işleme servisleri

| Dosya | Görevi |
|-------|--------|
| `llm.py` | ChatGPT API ile konuşma |
| `memory.py` | Önceki konuşmaları hatırlama |
| `message_engine.py` | Mesaj formatlama ve kontrol |
| `audio_processing.py` | Ses → Metin + konuşmacı ayırımı (Whisper + diarization) 🆕 |
| `emotion_analysis.py` | Duygu analizi (transformer modelleri) 🆕 |
| `referee_engine.py` | Kim haklı? NLP puanlama + karar 🆕 |
| `text_to_speech.py` | Sonucu sese çevir (opsiyonel) 🆕 |

---

### `utils/`
Yardımcı araçlar

| Dosya | Görevi |
|-------|--------|
| `config.py` | Ortam dosyalarını (.env) okuma |
| `logging.py` | Uygulama hataları ve günlük kayıtları |

---

### `main.py`
Uygulamanın ana giriş noktası (FastAPI burada başlar)

---

## 🔄 Servisler Arası Akış
Kullanıcı Ses Kaydı Yükler → [routes/audio.py]
↓
Ses Metne Dönüşür + Kişiler Ayrılır → [services/audio_processing.py]
↓
Duygular Analiz Edilir → [services/emotion_analysis.py]
↓
NLP & ChatGPT Yorumlar → [services/llm.py + referee_engine.py]
↓
Karar Verilir → ‘Ali daha yapıcıydı…’
↓
Sesli Yanıt Üretilir (Opsiyonel) → [text_to_speech.py]
↓
Kullanıcıya JSON + Ses Dosyası Döner

---

## 🧠 Kullanılan Teknolojiler

| Teknoloji | Açıklama |
|----------|----------|
| **FastAPI** | API sunucusu, hızlı ve modern |
| **OpenAI Whisper** | Ses → Metin çevirisi |
| **pyannote-audio** | Konuşan kişi ayırımı |
| **HuggingFace Transformers** | Duygu analizi modelleri |
| **OpenAI ChatGPT API** | LLM ile konuşma ve analiz |
| **TTS (gTTS, ElevenLabs, Coqui)** | Sonucu sesli oynatma (opsiyonel) |
| **Docker** | Uygulamanın taşınabilir şekilde çalışması |
| **.env** | Gizli API anahtarları, ayarlar |
| **Logging** | Hata ve analiz takibi |

---

## 🧪 Uygulama Akışı

| Aşama | Açıklama |
|-------|----------|
| 1 | Kullanıcı mikrofonla ses kaydı gönderir |
| 2 | Whisper ile metne çevrilir |
| 3 | pyannote ile konuşmacılar ayrılır (Ali vs. Ayşe) |
| 4 | Metinler ayrı ayrı duygu analizine gönderilir |
| 5 | LLM (ChatGPT) ile NLP analiz yapılır |
| 6 | Yapıcı/empatik gibi puanlamalarla kimin daha haklı olduğuna karar verilir |
| 7 | (İsteğe bağlı) Bu sonuç sesli olarak da kullanıcıya okunur |
| 8 | Sonuç ekranda gösterilir, istenirse kaydedilir veya paylaşılır |

---

## 📚 Öğrenilmesi Gerekenler

Bu projeyi geliştirmek için şunları bilmen faydalı olacaktır:

### 🔤 Temel:
- Python programlama
- FastAPI (REST API geliştirme)
- JSON yapısı
- Docker & .env yapısı

### 🤖 AI Bileşenleri:
- Whisper: Speech-to-Text
- pyannote-audio: Speaker diarization
- Transformers: Emotion analysis
- ChatGPT Prompt Engineering
- TTS: gTTS, Coqui, ElevenLabs

---

## ✅ Özet

**Hakemly**, yapay zeka destekli, ses tabanlı duygu analizine sahip, kimin haklı olduğunu objektif biçimde analiz eden benzersiz bir mobil/masaüstü uygulama altyapısıdır.

Yapılacaklar:
- [ ] API ve servis yapısını oturt
- [ ] Speech to text + konuşmacı ayır
- [ ] Emotion analysis & NLP
- [ ] ChatGPT ile karar ve öneri üret
- [ ] TTS ile geri bildirimi sesli ver (isteğe bağlı)
- [ ] UI/UX prototipi (mobil ya da web)

---

---

## 📂 Eklenen Dosyalar

- `src/Hakemly/tests/test_client.py` → API uç noktalarını manuel test etmek için.  
- `tasks.py` → Invoke görevleri (`run`, `test`, `install`, `clean`).  
- `src/Hakemly/services/text_to_speech.py` → Hugging Face tabanlı TTS servisi.  
- `src/Hakemly/routes/audio.py` → `/audio/*` endpointlerini yöneten FastAPI router.  
- `outputs/` → Test sonrası üretilen `.wav` dosyaları burada tutulur.  
---

## ▶️ Çalıştırma
invoke run
Servis varsayılan olarak http://127.0.0.1:8000 üzerinde çalışır.

Swagger dokümantasyonu: http://127.0.0.1:8000/docs

## 🧪 Test Etme
invoke test

test_client.py aşağıdaki endpointleri test eder:

/chat/echo → LLM’den JSON yanıt döner

/audio/chat_speak → LLM yanıtını ses dosyası (outputs/llm_reply.wav) olarak kaydeder

/audio/test → Basit test sesi (outputs/test.wav) üretir

## 🔌 API Endpointleri
POST /audio/chat_speak
Request:
{
  "message": "Merhaba!",
  "session_id": "s1"
}
Response:
Binary wav dosyası → outputs/llm_reply.wav

GET /audio/test
→ Basit bir test sesi üretir (outputs/test.wav).

---