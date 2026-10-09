# AI Assistant: Chat, Voice, Image and File Search

A full-stack AI assistant with four skills:

| Skill | What it does |
|---|---|
| **Chat** | Text conversation with streaming replies |
| **Voice** | Real-time spoken conversation (speech in, speech out) |
| **Image** | Upload or paste a photo and ask questions about it |
| **File** | Upload documents and get answers with numbered citations (RAG with Google Gemini File Search) |

It uses a small, efficient Gemini model by default and can optionally run local models through Ollama.

## Architecture

```
Browser (Next.js)
   |-- HTTP/SSE ----------> FastAPI API (:8000) --> Providers: Gemini (main), Ollama (optional)
   |                              |--> Library: Gemini File Search store (RAG)
   |                              '--> /api/voice/chat  (same chat pipeline, used by voice)
   '-- WebRTC audio --> LiveKit Cloud <--> Voice agent (speech-to-text, reply, text-to-speech)
```

- `app/web`: Next.js 16, React 19, TypeScript, Tailwind CSS 4, shadcn/ui
- `app/api`: FastAPI, Python 3.13, uv, ruff, pytest
- `app/agent`: LiveKit voice agent (Python)
- `.claude/skills`: the Claude Code skills used to build the app (`fullstack-ai-assistant`, `add-photos`, `add-files-library`, `livekit-voice-mode`)

## Requirements

- Node 24, Python 3.13 (uv can install it), [uv](https://docs.astral.sh/uv/)
- A Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey)
- A [LiveKit Cloud](https://cloud.livekit.io) project (only for voice)
- Optional: [Ollama](https://ollama.com) for local models (for example `gemma3:1b` for chat and `moondream` for images)

## Setup

**1. Create your private config files** (these are git-ignored; never commit them):

```
copy app\api\.env.example app\api\.env
copy app\agent\.env.example app\agent\.env
```

Edit `app/api/.env`:

```
GEMINI_API_KEY=your_key
GEMINI_MODEL=gemini-3.5-flash-lite
RAG_MODEL=gemini-3.5-flash-lite
LIBRARY_TOKEN=any_long_random_string
LIVEKIT_URL=wss://your-project.livekit.cloud
LIVEKIT_API_KEY=your_livekit_key
LIVEKIT_API_SECRET=your_livekit_secret
VOICE_AGENT_NAME=my-agent
```

In `app/agent/.env`, use the **same** `LIVEKIT_*` values and the same `VOICE_AGENT_NAME`.

Generate a token with: `python -c "import secrets; print(secrets.token_urlsafe(32))"`

Model names change over time. If one is rejected, pick a stable model from Google's current Gemini model list.

**2. Install and run** (three terminals):

```
# Terminal 1: API
cd app/api
uv sync
uv run fastapi dev app/main.py

# Terminal 2: web
cd app/web
npm install
npm run dev

# Terminal 3: voice agent (first time: download files)
cd app/agent
uv run python voice_agent.py download-files
uv run python voice_agent.py dev
```

Open http://localhost:3000. Choose the Gemini model in the model picker (not Auto) for images and files.

## How it works

- **Chat:** the UI sends the message and history to the API, which calls the selected provider and streams the reply back.
- **Image:** photos are validated and size-limited, then sent to a vision-capable model.
- **File / RAG:** documents are indexed in a Gemini File Search store (chunking and embeddings are handled by Gemini). Questions retrieve relevant chunks, and answers include numbered citations. Admin pages are under `/library`.
- **Voice:** the browser streams audio to LiveKit; the agent transcribes it, asks the API for a reply through the same chat pipeline, and speaks the answer.

## Tests and checks

```
cd app/api
uv run ruff check
uv run pytest        # 96 tests

cd app/web
npm run typecheck
npm run build
```

## Security notes

- API keys and secrets live only in `.env` files, which are git-ignored.
- Before any commit, run `git status` and make sure no `.env` file is listed.
- For a public deployment, also set `VOICE_AGENT_TOKEN` in both `.env` files and put the API behind login.

## Possible improvements

- Deploy with Docker Compose; run more agent workers; managed database and storage for the library
- Per-user libraries and permissions, login and rate limits
- More providers (OpenAI and Anthropic adapters are already in the code), multilingual voice
- Evaluation and analytics for answer quality

## Author

NOOR
