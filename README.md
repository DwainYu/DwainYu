# Hi, I'm Playmaker (DwainYu)

**AI Agent Engineering · Agent Tooling · AI Applications**

<sub>AI 智能体工程 · 智能体工具链 · AI 应用开发</sub>

I build AI applications and treat them as software: typed boundaries, tests that run offline, and
explicit failure paths instead of silent fallbacks. Most projects are local-first, with documented
tests and reproducible steps for claims that can be verified locally.

我把 AI 应用当作软件工程来做：类型边界、可离线运行的测试、明确的失败路径，而不是静默兜底。
多数项目采用本地优先设计，并为可在本地验证的结论提供测试或复现步骤。

- **AI Agent Engineering** — Agent Runtime, tool calling, RBAC at the tool boundary, auditable traces
- **Agent Tooling** — read-only tool facades, MCP surfaces, guardrails around small local LLMs
- **AI Applications** — RAG with citations and refusal, local-first products, Python backends
- **Exploring** — Agent Skills, CLI coding agents (Pi / Codex), RAG evaluation
- Everything below is either verified in a repository, or explicitly labelled as learning

---

## Featured Projects

### 1. Enterprise AI Platform Demo — a governed AI assistant, not a chatbot demo

**[github.com/DwainYu/enterprise-ai-platform-demo](https://github.com/DwainYu/enterprise-ai-platform-demo)** · FastAPI · React · SQLAlchemy · Chroma · Docker Compose

- **Problem:** in a company, the hard parts of an AI assistant are not "connecting a model" — they
  are *who may use which tool*, *what happens when the provider fails*, and *how you reconstruct
  what happened afterwards*.
- **What it does:** a Model Gateway that hides the provider behind an interface (business code never
  imports a provider SDK), heading-aware RAG that answers with `[n]` citations and refuses when
  evidence is missing, an Agent Runtime with `max_steps` / `max_tool_calls` caps, whitelist tools
  using fixed operations and parameterised SQL (the LLM never receives a table name), and RBAC
  enforced twice — at the route *and* inside `ToolExecutor`.
- **Engineering focus:** one `request_id` threaded through HTTP / Agent / Tool / Audit so a single
  chain explains an incident; a controlled error envelope that never leaks stack traces; provider
  failure returns a controlled 502 instead of silently degrading to mock.
- **Verified:** `uv run pytest -q` → **426 passed**, all offline with zero external dependencies;
  `ruff check` + `ruff format --check` + `tsc -b && vite build` green;
  `docker compose up --build` brings up API `:8000` and Web Console `:3000`.
- **Honest limits:** a **demo-grade prototype** — synthetic SQLite data, a fictional company corpus,
  three fixed bearer tokens (not JWT/SSO), mock embeddings by default.

### 2. TFT Log — a local-first training journal that closes a real feedback loop

**[github.com/DwainYu/tft-log](https://github.com/DwainYu/tft-log)** · TypeScript (strict) · React 19 · Vite 6 · Tailwind v4 · Dexie / IndexedDB · Vitest

- **Problem:** public TFT tools answer "what is strong now". They do not fix the actual gap — you
  remember the bad games, forget the good ones, and repeat the same mistakes next week.
- **What it does:** Quick Add (a whole match from keystroke `1` to saved in 1–2 minutes), decision
  logs with a hindsight verdict, a four-part post-game review that gates the `reviewed` state,
  autosaved review drafts, and a statistics dashboard that only shows what you recorded.
- **Engineering focus:** a pure, fixture-tested `domain/` layer (`[8,4,3,1]` → avg 4.0, Top-4 75%),
  `services/` owning the invariants, and `data/repository/*` as the only layer that touches Dexie.
- **Verified:** **531 tests** across 54 files (`vitest run`); live at
  **[dwainyu.github.io/tft-log](https://dwainyu.github.io/tft-log/)** and on **Bilibili Toy** — both
  verified reachable. A shipped read-only `agent-tools` facade (`get_recent_matches`,
  `get_common_mistakes`, `get_training_goals`, …) exposes the same data the UI reads, so a future
  coach agent needs no UI change.

### 3. md-content-publisher — a Markdown-to-platform publishing toolkit with a human gate

**[github.com/DwainYu/md-content-publisher](https://github.com/DwainYu/md-content-publisher)** · TypeScript · unified / remark / hast · Commander · React + Vite

- **Problem:** the same technical article has to become a generic HTML page, a WeChat-ready
  single-file product, and a Xiaohongshu script — each with its own hard constraints.
- **What it does:** Markdown → one content model → platform adapters (generic / WeChat /
  Xiaohongshu), plus an **Agent Mode** pipeline: rewrite → validate → human review → export. Export
  is refused unless `review.status === approved`, so an agent can never publish on its own.
- **Engineering focus:** deterministic guardrails around a small local LLM — code blocks and images
  may only be *referenced* from the source, tags are whitelisted, truncated JSON is repaired before
  parsing, and platform limits (Xiaohongshu title ≤ 20 chars, body ≤ 1000) are enforced by the
  validator rather than the prompt.
- **Verified:** **210 tests** across 21 files; the `mock` provider keeps the whole suite CI-safe
  without any model.

### 4. Laya Multilingual Playground — measuring a non-autoregressive decision model on CPU

**[github.com/DwainYu/laya-multilingual-playground](https://github.com/DwainYu/laya-multilingual-playground)** · Python 3.11 · PyTorch (CPU) · ModelScope · pytest

- **What it is:** a minimal reproducible experiment running **Laya Multilingual**, a
  non-autoregressive *System 1 decision model*, on WSL2 + CPU — one forward pass, three typed
  questions (`choice` / `score` / `noul`), no decoding loop and nothing to hallucinate.
- **The finding I wrote down:** `choice` decisions were usable (8/10 Chinese cases sensible), but
  the current checkpoint's zero-shot **`noul` behaviour is unreliable** — a pure refund ticket
  scored `noul = 0.9409`. The model card agrees: zero-shot typed-decision 0.342 against a 0.318
  random baseline, with uncalibrated temperatures. Recorded conclusion: *do not gate on `noul`
  before calibrating on your own data.*
- **Verified:** per-file SHA-256 against both ModelScope and Hugging Face; 13 tests; a latency log
  (cold start 20.5–22.6 s, warm ~129–141 ms) kept separate from the official T4 GPU figures, since
  the two are not comparable.

---

## What I'm Exploring · 正在探索

Learning and experimenting — not mastered, not production-proven.
正在学习和实验，尚未掌握、未经验证可用于生产：

- **Agent Skills** — reusable, scoped instructions for CLI agents instead of one-off prompts
- **CLI coding agents** (Pi, Codex) — how they change the shape of a delivery workflow
- **MCP** — exposing read-only tool surfaces an agent can call safely
- **RAG evaluation** — turning "it feels better" into a fixed probe set with recall@k and refusal
  accuracy
- **Non-autoregressive / System-1 models** — where a fast structured decision beats a generative one

---

## Technical Toolkit · 技术栈

| Area 领域 | Tools |
| --- | --- |
| **Languages & Backend** | Python (FastAPI, Pydantic v2, SQLAlchemy 2, uv, pytest, ruff), TypeScript (strict), some Go |
| **AI / Agent** | RAG (chunking, embeddings, citation, threshold calibration), Agent Runtime + tool calling, MCP, Ollama, ModelScope, Chroma |
| **Frontend & Data** | React 19, Vite, Tailwind CSS, Dexie / IndexedDB, Astro, Recharts, unified / remark / hast |
| **Dev Tools & Ops** | Git, GitHub Actions (CI + Pages), Docker Compose, nginx, WSL2, Playwright, vitest / Testing Library |

---

## Learning & Knowledge Sharing · 笔记与复盘

Notes and write-ups live inside the repositories, next to the code that proves them.
Representative examples:
笔记和复盘随仓库维护，与代码放在一起。代表性文档：

- [`docs/rag-threshold-calibration.md`](https://github.com/DwainYu/enterprise-ai-platform-demo/blob/main/docs/rag-threshold-calibration.md) —
  36 probes across two vector spaces, with the honest conclusion that a threshold alone cannot
  guarantee refusal
- [`IMPLEMENTATION_REPORT.md`](https://github.com/DwainYu/laya-multilingual-playground/blob/main/IMPLEMENTATION_REPORT.md) —
  versions, sizes, latency ranges and the gotchas actually hit, including a TensorFlow import deadlock
- [`ROADMAP.md`](https://github.com/DwainYu/tft-log/blob/main/ROADMAP.md) — phased scope with
  explicit non-goals and what is *not* implemented

---

## Contact / Links · 联系方式

Confirmed and verified reachable:

- Email: **[playmaker_ai@qq.com](mailto:playmaker_ai@qq.com)**
- GitHub: **[@DwainYu](https://github.com/DwainYu)**
- TFT Log demo: **[dwainyu.github.io/tft-log](https://dwainyu.github.io/tft-log/)**
- Blog (AstroPaper-based, still thin): **[my-blog](https://github.com/DwainYu/my-blog)**

---

<sub>Test counts and reported measurements are drawn from project records; experimental results are
labelled with their context. 测试数量与实测数据均依据项目记录；实验结果会注明对应的测试环境与背景。</sub>
