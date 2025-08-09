Below is a production-grade scaffold for RiftBrief (working name).  
The structure is purpose-built for “productivity-with-joy”: fast local dev, field-stitching to the back-end (call any provider you like), clean visual state-management, and a one-command CI to publish everywhere.

──────────────────
1. TL;DR DECISION LOG
──────────────────
Front-end   Flutter + Fluent UI → one code-base, desktop-grade keyboard shortcuts, blazing-hot-ui. (Left Tailwind rendering if you want to switch to web later.)
Language    Dart 3 + JSON-based contract for every step. No GraphQL boilerplate.
Back-end    Minimalist FastAPI for orchestration. Every endpoint is *a step*.  
State mgmt  flutter_riverpod (AsyncValue + codegen).  
Comms       WebSockets (back-pressure) + SSE for progress stream.  
Storage     SQLite local cache (drift), remote: Supabase if user wants cloud sync.  
Costs       Punch-out: add /pricing endpoint = transparent > shows $, tokens etc.

──────────────────
2. PROJECT LAYOUT
──────────────────
rift_brief/
├─ app                  – Flutter
│  ├─ lib/
│  │  ├─ models/        JSON → @freezed
│  │  ├─ providers/
│  │  ├─ services/      API & websocket layer
│  │  ├─ ui/            Screens & shared widgets
│  │  └─ main.dart
│  ├─ pubspec.yaml
│  └─ l10n.yaml         (i18n ready)
├─ server               – FastAPI websocket+SSE server (Python 3.11)
│  ├─ src/
│  │  ├─ api/           routers
│  │  ├─ chains/        LangSmith-compatible LLM chains
│  │  ├─ providers/     Perplexity, OpenAI, Claude … thin wrappers
│  │  └─ db/            SQLite + SQLModel
│  ├─ poetry.lock
│  └─ pyproject.toml
├─ prompts/             Prompt templates (.yaml) version-controlled
└─ TASKFILE.yml         One-command `task dev` spins front+back+db

──────────────────
3. QUICK START
──────────────────
# 1. Clone & install
git clone https://github.com/you/rift_brief.git && cd rift_brief
task setup   # installs Flutter, Python, pre-commit hooks, pulls .env template

# 2. Add secrets
echo "PERPLEXITY_API_KEY=*****" >> server/.env
echo "OPENAI_API_KEY=*****"   >> server/.env

# 3. Run everything in one terminal
task dev      # Flutter web+desktop live + FastAPI with reload + Drift dev db

──────────────────
4. SCHEMA SKETCH
──────────────────
Each run is “iterati” of WorkPhase enums. Async snapshots hand up to UI.
lib/models/work_run.dart
```dart
@freezed
class WorkRun with _$WorkRun {
  factory WorkRun({
    required String id,
    required String topic,
    required String audience,
    required double targetMinutes,
    required Map<String, dynamic> constraints,
    required List<WorkPhase> phases,
  }) = _WorkRun;

  factory WorkRun.fromJson(Map<String, dynamic> json) => _$WorkRunFromJson(json);
}

@freezed
class WorkPhase with _$WorkPhase {
  factory WorkPhase({
    required String stepName,                // “research”, “brief”, “outline”…
    required PhaseStatus status,             // pending | running | done | error
    required DateTime createdAt,
    required dynamic result,
    String? error,
  }) = _WorkPhase;
}
```

──────────────────
5. API CONTRACT
──────────────────
POST /runs               – create new WorkRun  
WS /runs/{id}/status     – server-push lifecycle updates  
POST /runs/{id}/phase/{step}/prompt – client override prompt / tweak  
GET  /runs/{id}/export   – returns zip with artifacts (json+txt+png)

──────────────────
6. FLUTTER HIGH-FI KEY SCREENS
──────────────────
1. Dashboard (Hero screen)  
   • Title panel: topic selector in command-bar (CMD+K) Fuzzy search, audience chips.  
   • Run cards = Live cards w/ phase progress pie + mini cost ticker.
2. Wizard modal (bottom sheet stage-craft)  
   • Horizontal stepper with CTA: “Apply tweak” → replaces prompt → shows diff-preview.  
   • One-click “Export” glides the card up and toast opens Finder/Files.
3. Markdown + side preview (bloc-style)  
   • Left live-editable mark-up, right auto-rendered thumbnails + citations.  
   • Magic “cite” button inserts `[#n]` inline (provider detected colour).

──────────────────
7. FLUTTER TASTY WIDGETS
──────────────────
• PhaseTimeline – animated row showing which steps ran & cost.  
• PromptField – monaco-style editable + instant diff vs canonical prompt.  
• CitationBubbleHero – flutter_markdown pragma renders as tiny card on hover.

──────────────────
8. FASTAPI HANDLER SAMPLE
──────────────────
server/src/api/research.py
```python
from fastapi import APIRouter, WebSocket, Depends
from schemas import ResearchRequest
from providers.perplexity import pplx_search

router = APIRouter(prefix="/research")

@router.post("/")
async def run_research(req: ResearchRequest, repo=Depends(RunRepo)):
    run = await repo.get(req.run_id)
    # fan-out across providers
    web_json = await pplx_search(req.query, n_results=10)
    return await repo.store_phase(
        run.id, "research", result=web_json
    )
```
──────────────────
9. PROMPT VERSIONING
──────────────────
prompts/research_brief.yaml
```
version: 2
description: "System prompt – step 3"
prompt: |
  You are a senior content strategist …  
```
Server loads on start = auto reload on file change (with uvicorn --reload).

──────────────────
10. LOCAL PRODUCTION PATH
──────────────────
`task build` builds:
• macOS .app  – dmg
• Windows    – .exe
• Linux      – .tar.gz
• Web        – static (hosted on Vercel/Firebase with /api fall-through).

──────────────────
11. PAUSED FOR YOUR GO/NO-GO
──────────────────
Open two new terminals today:
1. `cd app && flutter test` – pass 100% unit + golden.
2. `cd server && pytest`    – mocks providers.

Ready? Zero-crunch merge → tomorrow aligns on `nextStep`.
