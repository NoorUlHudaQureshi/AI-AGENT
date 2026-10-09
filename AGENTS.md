# AI-AGENT

## Stack
* **Web:** Node 24, Next.js 16, React 19, TypeScript 7, Tailwind CSS 4, shadcn/ui — `app/web/` :3000
* **API:** FastAPI, Python 3.13, uv, Ruff, Pytest — `app/api/` :8000
* **Local AI:** Ollama — `localhost:11434` (`gemma3:1b` chat, `moondream` vision)
* **Cloud AI:** Gemini (required: File Search RAG); others optional fallbacks
* **Skills:** `.claude/skills/` (fullstack-ai-assistant, add-photos, add-files-library, livekit-voice-mode)

## Commands
cd app/web && npm install && npm run dev
cd app/api && uv sync && uv run fastapi dev app/main.py
cd app/api && uv run pytest && uv run ruff check

## Rules
* Never commit `.env`, secrets, or API keys.
* Run tests after every change.
* Keep changes minimal and focused.
* Follow existing project patterns and conventions.
* Prefer type-safe, clean, maintainable code.
* Update tests when behavior changes.
* Do not introduce dependencies without a clear need.
* Read a skill's SKILL.md and review its changes before applying; use --dry-run first.