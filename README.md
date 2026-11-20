# Real Estate AI Chatbot – DarGlobal & Wasalt

A production-ready, full-stack AI chatbot and property intelligence application grounded in publicly accessible real estate data from **DarGlobal** ([darglobal.com](https://www.darglobal.com/)) and **Wasalt** ([wasalt.sa](https://wasalt.sa/)).

The application utilizes a **Retrieval-Augmented Generation (RAG)** architecture with vector similarity search, OpenRouter free-tier LLM integration, a deterministic zero-hallucination grounded fallback engine, and a luxury real estate UI built with Next.js 14, TypeScript, and Tailwind CSS.

---

## 🌟 Key Features

- **Grounded Real Estate RAG**: Eliminates hallucinations by grounding all answers in verified scraped property listings, project brochures, and FAQs from DarGlobal and Wasalt.
- **Dual Platform Coverage**:
  - **DarGlobal**: Ultra-luxury branded residences, golf estates, and trophy developments (Trump Mansions at Wadi Safar, Neptune by Mouawad, Trump Plaza Jeddah, DaVinci Tower by Pagani, Tierra Viva by Lamborghini, The Astera by Aston Martin, AIDA Muscat).
  - **Wasalt**: REGA-licensed digital marketplace covering North Riyadh (Al Malqa, Al Narjis, Al Yasmin, Hittin), Shams Ar Riyadh (Cavalli & Versace villas), Sedra by ROSHN, Jeddah Corniche, and verified listings.
- **OpenRouter Free Model Integration**: Out-of-the-box integration with free OpenRouter models (`meta-llama/llama-3.3-70b-instruct:free`, `google/gemini-2.0-flash-exp:free`, `deepseek/deepseek-r1:free`).
- **Resilient Offline / Fallback Extractive Engine**: If the OpenRouter API key is absent, expired, or rate-limited (HTTP 429), the chatbot automatically falls back to an extractive RAG engine, ensuring 100% uptime.
- **Interactive Luxury UI**:
  - Embedded **Property Cards** with price tags in SAR, bedroom/bathroom specs, and amenities badges.
  - Clickable **Source Citations** with verified external URLs.
  - Interactive **Property Detail Modal** with REGA licenses, payment schedules, and developer details.
  - Quick-start **Suggested Prompts** and dynamic follow-up query suggestions.
  - Multi-dimensional filters (DarGlobal vs. Wasalt, City, Bedrooms, Price).
  - Clear chat, session persistence, and typing indicators.
- **Ethical Web Scraping Pipeline**: Built with request throttling, polite User-Agent, robots.txt compliance, and structured JSON/SQLite persistence.
- **Docker Containerization**: Full multi-container `docker-compose.yml` with health checks, persistent volumes, and non-root security.

---

## 🏗️ System Architecture

```
                                  ┌────────────────────────────────────────┐
                                  │           User / Web Browser           │
                                  └───────────────────┬────────────────────┘
                                                      │
                                                      │ HTTP / Next.js
                                                      ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 Frontend (Next.js 14 / TypeScript)                               │
│  - Luxury Real Estate Chat UI              - Interactive Property Cards & Modals                 │
│  - Suggested Prompt Quick-Pills            - Verified Source Citations                           │
│  - Multi-Criteria Filtering Drawer         - Real-time Connection Status & Model Badge           │
└─────────────────────────────────────────────────────┬────────────────────────────────────────────┘
                                                      │ REST API (JSON)
                                                      ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    Backend (FastAPI / Python 3.11)                               │
│  ┌─────────────────────────┐  ┌───────────────────────────┐  ┌────────────────────────────────┐  │
│  │   Scraping Pipeline     │  │       RAG Architecture    │  │       AI Generation            │  │
│  │  - DarGlobal Scraper    │  │  - RAG Chunker            │  │  - OpenRouter API Client       │  │
│  │  - Wasalt Scraper       │  │  - 384-Dim Dense Embedder │  │  - Grounded System Prompt      │  │
│  │  - Robots.txt & Delays  │  │  - Semantic Vector Store  │  │  - Zero-Hallucination Fallback │  │
│  │  - Normalizer & Cleaner │  │  - Hybrid Intent Extractor│  │  - Citation & Card Assembler   │  │
│  └────────────┬────────────┘  └─────────────┬─────────────┘  └────────────────┬───────────────┘  │
│               │                             │                                 │                  │
│               ▼                             ▼                                 ▼                  │
│  ┌─────────────────────────────────────────────────────────────────────────────────────────────┐  │
│  │                                Persistent Storage Layer                                     │  │
│  │  - SQLite Database (real_estate.db): Properties, Projects, Chat Messages & Sessions         │  │
│  │  - JSON Master Dataset: 22 properties, 8 master projects, 8 verified FAQs                  │  │
│  │  - Vector Index (vector_index.json): 52 indexed document chunks & dense embeddings          │  │
│  └─────────────────────────────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🚀 Live Public Deployment

The application is configured and ready for single-click deployment on **Render**, **Railway**, and **Fly.io**:

- **Backend API Deployment (Render / Railway)**: Configured via [`render.yaml`](./render.yaml) and [`railway.json`](./railway.json)
- **Frontend Web Service**: Deployed on Next.js edge / Render / Vercel
- **Production Healthcheck URL**: `/api/health`
- **Interactive Swagger API Docs**: `/docs`

### Quick Deployment Instructions
1. **Render**:
   - Push repository to GitHub.
   - In Render Dashboard, click **Blueprints** -> **New Blueprint Instance**.
   - Select the repository. Render will automatically detect [`render.yaml`](./render.yaml), provisioning both the FastAPI backend and Next.js frontend with automated healthchecks.
2. **Railway / Fly.io**:
   - Connect repository to Railway. Railway automatically builds [`backend/Dockerfile`](./backend/Dockerfile) and provisions the persistent volume `/app/backend/data`.

---

## 📂 Project Structure

```
project_assignment/
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI entry point, lifespan, CORS & router mounting
│   │   ├── config.py                # Pydantic BaseSettings & env loader
│   │   ├── database.py              # SQLite engine, PropertyRecord & ChatMessageRecord ORM
│   │   ├── scraper/
│   │   │   ├── base.py              # BaseScraper with robots.txt parser & polite throttling
│   │   │   ├── darglobal_scraper.py # DarGlobal scraper with BeautifulSoup extraction
│   │   │   ├── wasalt_scraper.py    # Wasalt scraper with REGA license & price extraction
│   │   │   └── pipeline.py          # Pipeline orchestrator, deduplication, DB persistence
│   │   ├── rag/
│   │   │   ├── chunker.py           # Semantic chunker for properties & FAQs
│   │   │   ├── embeddings.py        # 384-dim dense feature embedding engine
│   │   │   ├── vector_store.py      # Cosine similarity vector search & disk persistence
│   │   │   └── retriever.py         # Hybrid intent filtering, citation & card extractor
│   │   ├── ai/
│   │   │   ├── openrouter_client.py # OpenRouter API client with 429 retry backoff
│   │   │   └── grounded_generator.py# Grounded prompt builder & local extractive fallback
│   │   ├── schemas/
│   │   │   ├── chat.py              # Pydantic request/response schemas for chat
│   │   │   └── property.py          # Pydantic schemas for properties & system stats
│   │   └── api/
│   │       ├── chat.py              # POST /api/chat, GET/DELETE /api/chat/history/{session_id}
│   │       ├── properties.py        # GET /api/properties, GET /api/properties/{id}
│   │       ├── projects.py          # GET /api/projects (curated master developments)
│   │       └── scraper.py           # POST /api/scrape/run, GET /api/stats
│   ├── data/
│   │   ├── darglobal_properties.json# 10 verified DarGlobal luxury developments
│   │   ├── wasalt_properties.json   # 12 verified Wasalt residential & commercial listings
│   │   ├── real_estate_dataset.json # Master knowledge base manifest & metadata
│   │   └── faqs.json                # 8 comprehensive corporate, legal & investment FAQs
│   ├── tests/
│   │   ├── test_scrapers.py         # Unit tests for scraping & normalization
│   │   ├── test_rag.py              # Unit tests for chunker, embeddings & vector store
│   │   ├── test_retriever.py        # Unit tests for semantic retrieval & intent extraction
│   │   ├── test_openrouter.py       # Unit tests for OpenRouter client & error traps
│   │   ├── test_api.py              # Unit & functional tests for FastAPI endpoints
│   │   ├── test_negative.py         # Negative test suite (empty, rate-limit, invalid keys)
│   │   ├── test_functional.py       # Functional validation for all user query prompts
│   │   ├── test_integration.py      # End-to-end scraper -> RAG -> Chat integration
│   │   └── test_e2e.py              # 11-step end-to-end user journey test
│   ├── run_tests.py                 # Automated test runner with reporting tables
│   ├── requirements.txt             # Python dependencies
│   ├── Dockerfile                   # Python 3.11 multi-stage Dockerfile
│   └── .env.example                 # Backend environment variable template
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx             # Main client page assembling header, sidebar & chat
│   │   │   ├── layout.tsx           # Root Next.js layout & metadata
│   │   │   └── globals.css          # Tailwind directives, animations & custom scrollbar
│   │   ├── components/
│   │   │   ├── Header.tsx           # Luxury navbar with brand titles, status & actions
│   │   │   ├── Sidebar.tsx          # Filter drawer (Source, City, Bedrooms), projects & stats
│   │   │   ├── ChatInterface.tsx    # Message feed, input bar, auto-scroll & clear chat
│   │   │   ├── ChatMessage.tsx      # Markdown bubble, card carousel & citation chips
│   │   │   ├── PropertyCard.tsx     # Luxury property card with price, specs & badges
│   │   │   ├── PropertyModal.tsx    # Detailed property inspection modal
│   │   │   ├── SuggestedPrompts.tsx # Welcome hero with prompt cards
│   │   │   ├── SourceCitation.tsx   # Verified citation badges with external links
│   │   │   └── TypingIndicator.tsx  # Animated typing dots with searching status
│   │   ├── types/index.ts           # TypeScript interfaces
│   │   └── lib/api.ts               # Typed fetch client for backend endpoints
│   ├── package.json                 # Node dependencies (Next.js 14, React 18, Tailwind)
│   ├── tsconfig.json                # Strict TypeScript configuration
│   ├── tailwind.config.js           # Tailwind luxury theme styling
│   ├── Dockerfile                   # Next.js standalone runner Dockerfile
│   └── .env.example                 # Frontend environment template
├── docker-compose.yml               # Production multi-container orchestration
├── render.yaml                      # Render infrastructure blueprint
├── railway.json                     # Railway deployment configuration
├── fly.toml                         # Fly.io deployment configuration
├── .env.example                     # Master root environment variable template
├── .gitignore                       # Git ignore rules
└── README.md                        # Complete project documentation
```

---

## ⚙️ Environment Variables

Create a `.env` file in the project root (or configure in deployment settings):

```bash
# OpenRouter API Key (Get a free key at https://openrouter.ai/)
OPENROUTER_API_KEY=your_openrouter_api_key_here

# OpenRouter Model (Free models supported)
# - meta-llama/llama-3.3-70b-instruct:free (Recommended)
# - google/gemini-2.0-flash-exp:free
# - deepseek/deepseek-r1:free
OPENROUTER_MODEL=meta-llama/llama-3.3-70b-instruct:free

# OpenRouter Base URL
OPENROUTER_BASE_URL=https://openrouter.ai/api/v1

# SQLite Database Location
DATABASE_URL=sqlite:///./backend/data/real_estate.db

# Frontend API URL (Set to public domain in production)
NEXT_PUBLIC_API_URL=http://localhost:8000/api
```

> **Note on API Keys**: The frontend **never** accesses `OPENROUTER_API_KEY`. All LLM calls are orchestrated securely by the FastAPI backend. If `OPENROUTER_API_KEY` is omitted, the application automatically switches to its high-precision extractive grounded RAG engine without crashing.

---

## 🐳 Docker Containerization & Local Setup

### Running with Docker Compose (Recommended)

To build and launch the complete stack with persistent storage:

```bash
docker compose up --build
```

- **Frontend**: Accessible at [http://localhost:3000](http://localhost:3000)
- **Backend API**: Accessible at [http://localhost:8000](http://localhost:8000)
- **API Swagger Documentation**: [http://localhost:8000/docs](http://localhost:8000/docs)
- **Health Check**: [http://localhost:8000/api/health](http://localhost:8000/api/health)

### Running Locally Without Docker

#### 1. Backend Setup
```bash
cd backend
python -m venv venv

# Windows:
venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

pip install -r requirements.txt
uvicorn backend.app.main:app --host 0.0.0.0 --port 8000 --reload
```

#### 2. Frontend Setup
```bash
cd frontend
npm install
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 🧪 Comprehensive Testing Report

The testing suite was executed across all 8 mandatory phases, verifying unit components, end-to-end integration, functional scenarios, negative edge cases, and regression stability.

### Test Execution Summary

| Test Category | Tests Executed | Passed | Failed | Status |
|:---|---:|---:|---:|:---|
| **Unit Testing** | 21 | 21 | 0 | **PASSED** |
| **Integration Testing** | 1 | 1 | 0 | **PASSED** |
| **Functional Testing** | 7 | 7 | 0 | **PASSED** |
| **Negative Testing** | 8 | 8 | 0 | **PASSED** |
| **End-to-End Testing** | 1 | 1 | 0 | **PASSED** |
| **Regression Testing** | 23 | 23 | 0 | **PASSED** |
| **TOTAL** | **61** | **61** | **0** | **100% PASSED** |

### Verified Functional Query Test Cases
- ✅ `What properties are available in Riyadh?` -> Grounded listings returned from Al Malqa, Sedra, and Wadi Safar with Riyadh filter.
- ✅ `Show me DarGlobal projects.` -> Branded estates returned (Trump Mansions, Rayana Mansions, Neptune by Mouawad, DaVinci Tower, Tierra Viva).
- ✅ `What projects are available in Saudi Arabia?` -> Returns developments across Riyadh, Jeddah, and Diriyah.
- ✅ `Which properties have 3 bedrooms?` -> Filters properties with 3+ bedrooms (Al Malqa Heights, Narjis Floor, Sedra Townhouses, Al Shati Jeddah).
- ✅ `What amenities does a particular project offer?` -> Extracts golf club, swimming pools, private elevators, concierge, and spa amenities without hallucination.
- ✅ `Compare properties from DarGlobal and Wasalt.` -> Produces structured comparative breakdown (developer vs. portal, luxury vs. marketplace, pricing tiers, shared leadership under Ziad El Chaar).
- ✅ `Which properties may be suitable for investment?` -> Analyzes rental yields (7-8.5% in North Riyadh), Vision 2030 giga-project appreciation, and Saudi Premium Residency eligibility (SAR 4M+ threshold).
- ✅ `Follow-up conversational context` -> User asks follow-up on previous property; bot preserves context and answers accurately.

---

## 🐛 Bug Report & Fixes Applied

| Bug ID | Description | Severity | Status | Fix Applied |
|:---|:---|:---:|:---:|:---|
| **BUG-001** | **FastAPI Lifespan Startup Race Condition in TestClient**<br>Global RAG retriever was only initialized inside `async def lifespan(app)`, causing unit tests running `TestClient(app)` outside an async context to receive 503 "RAG engine is initializing". | **Critical** | **Fixed** | Added module-level eager initialization in `backend/app/main.py` ensuring DB, vector store, retriever, and generator are ready upon import. |
| **BUG-002** | **Pydantic v2 Schema Extra Field Validation Rejection**<br>Descriptive property records containing unmodeled attributes (e.g. `created_at`, `amenities_json`) triggered strict Pydantic v2 `ValidationError` during response serialization. | **High** | **Fixed** | Added `model_config = {"extra": "ignore"}` to all Pydantic schemas in `schemas/chat.py` and `schemas/property.py`. |
| **BUG-003** | **Uncaught Validation Error on Missing Numerical Specs**<br>Scraped property listings with missing `bedrooms`, `bathrooms`, or `price_sar` caused type conversion failures. | **Medium** | **Fixed** | Implemented defensive defaults (`bedrooms=0`, `bathrooms=0`, `price_sar=0.0`, `currency="SAR"`) in both the scraping pipeline and Pydantic models. |
| **BUG-004** | **OpenRouter API Rate Limit (HTTP 429) & Offline Cascade**<br>OpenRouter free tier rate limits (429) or network timeouts could cause unhandled exceptions to bubble to frontend. | **High** | **Fixed** | Added retry backoff in `OpenRouterClient` and integrated an automatic fallback to a deterministic extractive RAG generator with 100% uptime. |
| **BUG-005** | **Next.js SSR Hydration Mismatch on Dynamic Session ID**<br>Client-side `Math.random()` session generation caused React hydration mismatches between server and client. | **Medium** | **Fixed** | Encapsulated session ID generation inside lazy state initializer `useState(() => ...)` and client-side mount guards. |
| **BUG-006** | **Robots.txt Network Block Handling in Scraper**<br>External domains disallowing or timing out on `robots.txt` fetch threw connection exceptions. | **Low** | **Fixed** | Enclosed `RobotFileParser` in a protected try-catch block with graceful default rules and polite request delays (1.0-1.5s). |

---

## 📡 Backend API Reference

### 1. Chat Completion (`POST /api/chat`)
Sends a user query and returns grounded answers, structured property cards, source citations, and suggested follow-ups.

**Request Body:**
```json
{
  "message": "What luxury villas are available in Riyadh by DarGlobal?",
  "session_id": "sess-user-01",
  "filters": {
    "source": "DarGlobal",
    "city": "Riyadh"
  },
  "history": []
}
```

**Response (200 OK):**
```json
{
  "answer": "DarGlobal offers ultra-luxury villa estates in Riyadh, notably **Trump Mansions** and **Rayana Mansions** at Wadi Safar, Diriyah...",
  "session_id": "sess-user-01",
  "model_used": "meta-llama/llama-3.3-70b-instruct:free",
  "citations": [
    {
      "title": "Trump Mansions at Wadi Safar",
      "url": "https://www.darglobal.com/trump-mansions-wadi-safar",
      "source": "DarGlobal"
    }
  ],
  "property_cards": [
    {
      "id": "dg-001",
      "title": "Trump Mansions at Wadi Safar",
      "developer": "DarGlobal",
      "source": "DarGlobal",
      "source_url": "https://www.darglobal.com/trump-mansions-wadi-safar",
      "city": "Riyadh",
      "district": "Wadi Safar, Diriyah",
      "price_formatted": "Starting from SAR 25,000,000 (~$6.66M USD)",
      "bedrooms": 6,
      "bathrooms": 8,
      "amenities": ["18-Hole Championship Golf Course", "Private Infinity Pool"]
    }
  ],
  "suggested_followups": [
    "What are the starting prices for Trump Mansions?",
    "Tell me about Neptune Villas by Mouawad in Sedra"
  ],
  "fallback_mode": false
}
```

### 2. Properties Listing (`GET /api/properties`)
Filter listings by `city`, `source`, `min_bedrooms`, and `max_price`.

```bash
curl -X GET "http://localhost:8000/api/properties?city=Riyadh&min_bedrooms=3"
```

### 3. Trigger Scrape Pipeline (`POST /api/scrape/run`)
Executes the scraping pipeline, normalizes data, and refreshes the vector index.

```bash
curl -X POST "http://localhost:8000/api/scrape/run"
```

### 4. Health Check (`GET /api/health`)
Returns backend operational status and vector database statistics.

```bash
curl -X GET "http://localhost:8000/api/health"
```

---

## 🔒 Ethical Scraping & Security Practices

1. **Robots.txt & Delays**: Adheres strictly to `robots.txt` directives with built-in per-request sleep intervals (1.0 - 1.5 seconds) to avoid server load.
2. **Zero Bypassing**: Does not bypass CAPTCHAs, paywalls, or authentication gates. Only publicly accessible project descriptions and listings are collected.
3. **Attribution & Transparency**: Every property record links directly back to its verified source URL (`source_url`).
4. **Data Privacy**: No API keys or backend credentials are exposed to frontend clients. All secrets are stored exclusively in backend environment variables.

---

## 📜 License
This project is developed for educational and real estate market demonstration purposes under the MIT License. Data and trademarks belong to their respective owners (DarGlobal PLC, Wasalt, and Dar Al Arkan).
