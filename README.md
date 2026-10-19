# CodeGraph

**An agent that reasons over your codebase as a graph.**
Built for [WeMakeDevs x FalkorDB "Graph Hacks"](https://www.wemakedevs.org/hackathons/falkordb)
— Track 01: *Agents That Act on Connected Data*.

CodeGraph ingests a Python repository (files, functions, classes, commits,
authors) into [FalkorDB](https://falkordb.com) as a property graph, then lets
an LLM agent answer real questions for three kinds of people:

- **A new programmer** reading unfamiliar code: *"Where should I start
  reading?"*, *"Explain this function in plain English"*, *"How risky is
  this change?"*
- **An open-source contributor** deciding whether to get involved: *"Is this
  project still maintained?"*, *"Who should I tag for review?"*, *"What's a
  safe first PR?"*
- **A new teammate** joining a codebase: *"What's changed recently?"*,
  *"Who owns this file?"*, *"Which files have a bus factor of 1?"*

...by calling graph-native tools (blast-radius, PageRank, recent-activity
analysis, ownership/bus-factor queries) over a real **MCP** server — and
shows you the exact subgraph and Cypher query behind every answer, so the
reasoning is traceable, not a black box. A zero-AI-cost "Start Here" card
(built from the same graph tools, no Gemini call) also orients a new visitor
the moment the page loads.

**What makes this different from a"code Q&A" tool (including
FalkorDB's own [code-graph](https://github.com/FalkorDB/code-graph) demo):
memory.** Click "📌 Save as team insight" on any answer, and it's written
back into the graph as an `:Insight` node linked to the exact code it's
about. Ask about that same function again — next session, next day, after a
restart — and the agent recalls and quotes the prior note before answering.
The graph gets smarter the more a team uses it, instead of being a frozen
snapshot from ingestion time.

## Why this matters for the track

| Judging criterion                  | How CodeGraph addresses it |
|--------------------------------------|-------------------------------|
| Multi-hop reasoning                  | `blast_radius`/`shortest_path` chain with other tools across multiple agent steps |
| Graph queries/algorithms as tools    | PageRank, variable-length traversals, all exposed as callable tools |
| Tool selection through MCP           | A real MCP server + MCP client (stdio subprocess), not simulated |
| Actions based on what the graph shows| The agent picks which tool(s) to call based on the question |
| Explainable results                  | Every answer ships with the literal Cypher + the nodes/edges used |

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full design.

## Quickstart

```powershell
# 1. Python side (ingestion + MCP server + backend share one env)
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt

# 2. Copy and fill in .env (FalkorDB Cloud + Gemini credentials)
copy .env.example .env

# 3. Ingest a small PUBLIC Python repo (never point this at proprietary code)
python -m ingestion.ingest --repo https://github.com/psf/requests.git

# 4. Backend (also spawns the MCP server as a subprocess)
uvicorn backend.main:app --reload --port 8000

# 5. Frontend
cd frontend
npm install
npm run dev
```

Open `http://localhost:5173` and start asking questions.

**Never run this against proprietary/company code** — the repo's contents
are sent to two third-party services (FalkorDB Cloud and Gemini).

## Project layout

```
common/        shared FalkorDB connection helper
ingestion/     parses a Python repo (ast + git history) into the graph
mcp_server/    the agent's tools, exposed over MCP (tools.py + server.py)
backend/       FastAPI app: MCP<->Gemini bridge + agent loop + /api/*
frontend/      React app: chat + graph visualization + Cypher trace panel
docs/
  ARCHITECTURE.md      full system design, mapped to judging criteria
  BUILD_GUIDE.md       step-by-step guide to rebuild this yourself, with explanations
  FALKORDB_LEARNING.md Cypher cheat sheet + how to start contributing to FalkorDB
```

## Learn more 

This project was deliberately documented as a teaching artifact, not just a
submission:

- [docs/BUILD_GUIDE.md](docs/BUILD_GUIDE.md) — rebuild the whole thing from
  scratch, step by step, with every concept (Cypher, MCP, Gemini function
  calling, the React graph view) explained as you go.
- [docs/TYPE_IT_YOURSELF.md](docs/TYPE_IT_YOURSELF.md) — the complete,
  literal version: every file's full content, in order, with beginner-level
  explanations of Python/JavaScript/CSS/Cypher basics, for typing out by
  hand with no coding background assumed.
- [docs/FALKORDB_LEARNING.md](docs/FALKORDB_LEARNING.md) — a FalkorDB/Cypher
  crash course, plus a concrete path toward contributing to FalkorDB as an
  open-source project afterward.
- [docs/BLOG_POST.md](docs/BLOG_POST.md) — a draft blog post about this
  project (for the "Best blog post" side prize).

## Known limitations

- The call graph is resolved by simple name matching (not full type
  inference) — see the "Known limitations" section in
  [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the full list.
- Only Python repos are parsed in this version.
