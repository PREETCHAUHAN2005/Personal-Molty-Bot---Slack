<div align="center">

```
 ██████╗ ███╗   ███╗    ██████╗  ██████╗ ████████╗
██╔═══██╗████╗ ████║    ██╔══██╗██╔═══██╗╚══██╔══╝
██║   ██║██╔████╔██║    ██████╔╝██║   ██║   ██║   
██║   ██║██║╚██╔╝██║    ██╔══██╗██║   ██║   ██║   
╚██████╔╝██║ ╚═╝ ██║    ██████╔╝╚██████╔╝   ██║   
 ╚═════╝ ╚═╝     ╚═╝    ╚═════╝  ╚═════╝    ╚═╝   
```

### *Your team's second brain — living inside Slack.*

**Recall anything. Connect everything. Never lose context again.**

<br/>

[![Node.js](https://img.shields.io/badge/Node.js-18%2B-3C873A?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org/)&nbsp;
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)&nbsp;
[![Slack](https://img.shields.io/badge/Slack-Bolt.js-4A154B?style=flat-square&logo=slack&logoColor=white)](https://slack.dev/bolt-js/)&nbsp;
[![OpenAI](https://img.shields.io/badge/GPT--4o-Powered-412991?style=flat-square&logo=openai&logoColor=white)](https://openai.com/)&nbsp;


<br/>

> *"What did we decide about the API last Tuesday?"*
> *"Create GitHub issues from every bug we discussed this week."*
> *"Search Notion for the Q4 roadmap and summarize it."*
>
> OM Bot handles all of it — and remembers for next time.

</div>

---

##  What's Inside

- [The Pitch](#-the-pitch)
- [Three Superpowers](#-three-superpowers)
- [System Architecture](#-system-architecture)
- [Message Lifecycle — A Full Walkthrough](#-message-lifecycle--a-full-walkthrough)
- [Deep Dives](#-deep-dives)
  - [RAG — Slack as a Knowledge Base](#1-rag--slack-as-a-knowledge-base)
  - [Memory — It Knows You](#2-memory--it-knows-you)
  - [MCP — The Tool Layer](#3-mcp--the-tool-layer)
- [Setup & Installation](#-setup--installation)
- [Environment Reference](#-environment-reference)
- [See It In Action](#-see-it-in-action)
- [Full Tool Catalog](#-full-tool-catalog)
- [Project Map](#-project-map)
- [Troubleshooting Playbook](#-troubleshooting-playbook)
- [Contributing](#-contributing)

---

##  The Pitch

Most bots forget everything the moment a conversation ends. Most search tools are keyword-only. Most integrations require jumping between five tabs.

**OM Bot is different.** It combines three production-grade systems into one coherent intelligence layer for your Slack workspace:

| What | How | Why It Matters |
|:---|:---|:---|
|  **Semantic Search** | Vector embeddings over all indexed Slack messages | Finds *meaning*, not just keywords |
|  **Persistent Memory** | mem0.ai extracts facts from every conversation | Learns your preferences & context over time |
|  **Live Tool Access** | 59 tools across Slack, GitHub & Notion via MCP | Acts on the world, not just talks about it |

```
A standard LLM: ──────────────────────────────▶  response (stateless, isolated)

  OM Bot flow:  message
                  │
                  ├─▶ recall past memories ──────────────────────────────────┐
                  ├─▶ retrieve relevant Slack history (RAG) ─────────────────┤
                  ├─▶ load session context ─────────────────────────────────┤
                  │                                                          ▼
                  └──────────────────────────────▶  GPT-4o + 59 tools  ──▶ action
                                                                            │
                                                              ┌─────────────┘
                                                              ▼
                                                      store new memories
```

---

##  Three Superpowers

### RAG — Retrieval Augmented Generation
Every message sent in your channels is quietly indexed in the background. When you ask about something discussed in the past, OM Bot doesn't guess — it *retrieves* the actual conversation and builds its answer on real data.

- Background channel indexing every 60 minutes
- Semantic similarity scoring (not keyword matching)
- Works even when the bot doesn't have direct channel access

###  Long-Term Memory
After every conversation, OM Bot extracts facts and stores them. Next session, it already knows your GitHub handle, your team's preferences, your project names. The more you use it, the smarter it gets *about you*.

- User-controlled: view, add, or wipe memories anytime
- Powered by mem0.ai cloud
- Fully cross-session — nothing is lost between conversations

###  MCP Integration (GitHub + Notion)
Using Anthropic's Model Context Protocol, OM Bot connects to external services over a standardized interface. It doesn't just answer — it creates issues, reads files, searches databases, and updates pages.

- **26 GitHub tools** — repos, issues, PRs, code search, file reads
- **21 Notion tools** — search, read, create, query databases
- Extensible: add any MCP-compatible server with a few lines of config

---

##  System Architecture

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                            YOUR SLACK WORKSPACE                             ║
║                                                                              ║
║   ┌───────────┐   ┌───────────┐   ┌────────────┐   ┌───────────────────┐   ║
║   │ #general  │   │ #dev-team │   │    DMs     │   │  @om-bot (mention)│   ║
║   └─────┬─────┘   └─────┬─────┘   └─────┬──────┘   └─────────┬─────────┘   ║
╚═════════╪═══════════════╪═══════════════╪════════════════════╪═════════════╝
          └───────────────┴───────────────┴────────────────────┘
                                      │
                                      ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║               SLACK BOLT.JS  ·  Socket Mode Event Router                    ║
╚══════════════════════════════════════════════════════════════════════════════╝
                                      │
                                      ▼
╔══════════════════════════════════════════════════════════════════════════════╗
║                            OM BOT  AGENT CORE                                ║
║                                                                              ║
║  ┌──────────────────────────────────────────────────────────────────────┐    ║
║  │                       CONTEXT ASSEMBLY ENGINE                        │    ║
║  │  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────────────┐  │    ║
║  │  │ MEMORY CONTEXT  │  │   RAG CONTEXT   │  │   SESSION HISTORY    │  │    ║
║  │  │                 │  │                 │  │                      │  │    ║
║  │  │ "User is a      │  │ "On Oct 5, team │  │  Last 10 messages    │  │    ║
║  │  │  co-founder..." │  │  discussed..."  │  │  in current thread   │  │    ║
║  │  └─────────────────┘  └─────────────────┘  └──────────────────────┘  │    ║
║  └──────────────────────────────────────────────────────────────────────┘    ║
║                                      │                                       ║
║                                      ▼                                       ║
║  ┌──────────────────────────────────────────────────────────────────────┐    ║
║  │                    59 TOOLS AVAILABLE TO GPT-4o                      │    ║
║  │                                                                       │    ║
║  │  ┌─────────────────┐  ┌──────────────────┐  ┌─────────────────────┐  │    ║
║  │  │  SLACK  (12)    │  │  GITHUB  (26)    │  │  NOTION  (21)       │  │    ║
║  │  │  ─────────────  │  │  ──────────────  │  │  ─────────────────  │  │    ║
║  │  │  search_kb      │  │  create_issue    │  │  search             │  │    ║
║  │  │  send_message   │  │  list_repos      │  │  get_page           │  │    ║
║  │  │  get_history    │  │  get_file        │  │  query_database     │  │    ║
║  │  │  schedule       │  │  list_PRs        │  │  create_page        │  │    ║
║  │  │  memory ops     │  │  search_code     │  │  update             │  │    ║
║  │  │  + 7 more       │  │  + 21 more       │  │  + 16 more          │  │    ║
║  │  └─────────────────┘  └──────────────────┘  └─────────────────────┘  │    ║
║  └──────────────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════════════════════════════════════════════════╝
          │                         │                           │
          ▼                         ▼                           ▼
  ┌───────────────┐       ┌──────────────────┐       ┌──────────────────────┐
  │  ChromaDB     │       │   mem0  CLOUD    │       │    MCP  SERVERS      │
  │  (local)      │       │                  │       │                      │
  │               │       │  User memories,  │       │  ┌────────────────┐  │
  │  254 indexed  │       │  preferences &   │       │  │  GitHub MCP    │  │
  │  Slack msgs   │       │  facts stored    │       │  └────────────────┘  │
  │               │       │  per user_id     │       │  ┌────────────────┐  │
  │  OpenAI       │       │                  │       │  │  Notion  MCP   │  │
  │  text-embed-3 │       │  gpt-4o-mini     │       │  └────────────────┘  │
  └───────────────┘       └──────────────────┘       └──────────────────────┘
```

---

##  Message Lifecycle — A Full Walkthrough

Let's trace a single message from arrival to response.

**Scenario:** *"Search Slack for bugs we discussed, then create GitHub issues for each one."*

---

**① Event Arrives**
```
Slack Bolt.js catches the message event.
→ Is this a DM? Is the user approved? Is the bot mentioned?
→ React with 😊 to confirm receipt
→ Retrieve or create a session for conversation continuity
```

---

**② Memory Recall**
```
Query mem0 for anything relevant to this user + this message.

Retrieved:
  · "User's GitHub username is PREETCHAUHAN2005"
  · "User prefers detailed technical explanations"
  · "User is Preet Chauhan"
```

---

**③ RAG Pre-Check**
```
Does the message reference past discussions?
Keywords matched: "discussed", "bugs"

→ Trigger vector search across 254 indexed Slack messages
→ Query: "bugs"
→ Top 5 results returned with relevance scores (latency: 384ms)
```

---

**④ Context Assembly**
```
System prompt is built with layered context:

┌──────────────────────────────────────────────────────────────┐
│  [SYSTEM]  You are OM Bot, a helpful AI assistant...         │
├──────────────────────────────────────────────────────────────┤
│  [SYSTEM]  ## What I Remember About You                      │
│            1. GitHub: PREETCHAUHAN2005                             │
│            2. Prefers detailed explanations                  │
├──────────────────────────────────────────────────────────────┤
│  [SYSTEM]  ## Relevant Slack History                         │
│            [Oct 5] @john: Found a bug in login flow...       │
│            [Oct 7] @jane: The API timeout is critical...     │
├──────────────────────────────────────────────────────────────┤
│  [USER]    (prior turns in session)                          │
├──────────────────────────────────────────────────────────────┤
│  [USER]    "Search Slack for bugs, create GitHub issues."    │
└──────────────────────────────────────────────────────────────┘

+ 59 tool definitions attached to the API call
```

---

**⑤ LLM Reasons & Calls Tools**
```
Round 1 — GPT-4o decides to search first:
  CALL  search_knowledge_base({ query: "bugs", limit: 10 })
  ─────────────────────────────────────────────────────────
  RESULT  10 relevant Slack messages returned

Round 2 — Now creates the GitHub issues:
  CALL  github_create_issue({
          owner: "PREETCHAUHAN2005", repo: "openclaw",
          title: "Fix login timeout bug",
          body: "As discussed on Oct 5 in #dev-team..."
        })
  CALL  github_create_issue({ ... second issue ... })
  ──────────────────────────────────────────────────
  RESULT  Issue #42 created · Issue #43 created

Round 3 — Final answer composed (no more tool calls)
```

---

**⑥ Background Memory Save**
```
After response is sent (async, non-blocking):
→ mem0 analyzes the full conversation
→ Extracts: "User tracks bugs found in Slack discussions"
→ Stored for future sessions
```

---

**⑦ Response Delivered**
```
→ Remove 😊 reaction
→ Post formatted response to Slack
→ Thread if response is long or a thread already exists

"Found 10 bug discussions in Slack. Created 2 GitHub issues:
  · #42 — Fix login timeout bug
  · #43 — API response caching issue"
```

---

##  Deep Dives

### 1. RAG — Slack as a Knowledge Base

RAG turns your workspace history from a search-box problem into a *semantic* problem. The bot doesn't match keywords; it matches meaning.

#### Indexing Phase *(runs every 60 min in background)*

```
Slack Channels  ──▶  Message Extractor  ──▶  OpenAI Embeddings  ──▶  ChromaDB
(#general,           (text, user,             (text-embedding-        (254 docs,
 #dev-team,           timestamp,               3-small,                local disk)
 #random...)          channel, thread)         1536 dimensions)
```

#### Query Phase *(on every relevant message)*

```
User Query  ──▶  Same Embedding Model  ──▶  Cosine Similarity Search  ──▶  Top-N Results
                                             (across all 254 docs)          (ranked by score)
```

#### RAG Settings

| Parameter | Default | Purpose |
|:---|:---:|:---|
| `RAG_ENABLED` | `true` | Toggle the entire system |
| `RAG_EMBEDDING_MODEL` | `text-embedding-3-small` | OpenAI model for vectors |
| `RAG_MAX_RESULTS` | `10` | Documents returned per query |
| `RAG_MIN_SIMILARITY` | `0.3` | Score threshold (0 = anything, 1 = exact) |
| `RAG_INDEX_INTERVAL_MINUTES` | `60` | How often channels are re-indexed |

**Files:** `src/rag/vectorstore.ts` · `src/rag/embeddings.ts` · `src/rag/indexer.ts` · `src/rag/retriever.ts`

---

### 2. Memory — It Knows You

Most AI assistants have amnesia. Every session starts from zero. OM Bot is different — mem0 automatically extracts facts from your conversations and resurfaces them later as relevant context.

#### How Facts Get Stored

```
Conversation ends
       │
       ▼
gpt-4o-mini analyzes exchange for extractable facts
       │
       ▼
Stored in mem0 cloud, keyed to your Slack user_id
       │
       ▼
Semantically searchable on your next message
```

#### Memory Categories

| Category | Example Stored Fact | Effect on Next Session |
|:---|:---|:---|
| **Identity** | "User is Preet Chauhan" | Context-aware responses |
| **Technical prefs** | "User's GitHub username is PREETCHAUHAN2005" | Auto-fills tool arguments |
| **Style prefs** | "User wants detailed explanations" | Adjusts response depth |
| **Projects** | "User is building nano-kimi" | Understands domain context |
| **Interests** | "User cares about SOP and LOR processes" | Prioritizes relevant topics |

#### User Commands

```
"What do you remember about me?"      →  get_my_memories
"Remember that I prefer Python"       →  remember_this
"Forget my old project details"       →  forget_about
"Wipe everything you know about me"   →  forget_everything
```

---

### 3. MCP — The Tool Layer

MCP (Model Context Protocol) is Anthropic's open standard for connecting LLMs to external tools. Instead of brittle, hardcoded API integrations, MCP gives OM Bot a plug-and-play interface to any compatible service.

#### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       OM BOT PROCESS                            │
│                                                                 │
│   MCP CLIENT  (src/mcp/client.ts)                               │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  · Spawns MCP server child processes on startup         │   │
│   │  · Discovers all available tools via tools/list         │   │
│   │  · Routes tool calls over JSON-RPC (stdio)              │   │
│   │  · Converts MCP tool schemas → OpenAI function format   │   │
│   └─────────────────────────────────────────────────────────┘   │
│              │  stdio                      │  stdio              │
│              ▼                             ▼                     │
│   ┌──────────────────────┐   ┌───────────────────────────┐      │
│   │  GITHUB MCP SERVER   │   │   NOTION MCP SERVER       │      │
│   │  npx @mcp/server-    │   │   npx @notionhq/notion-   │      │
│   │  github              │   │   mcp-server              │      │
│   │  26 tools            │   │   21 tools                │      │
│   └──────────┬───────────┘   └────────────┬──────────────┘      │
└──────────────┼───────────────────────────┼─────────────────────┘
               ▼                           ▼
         api.github.com             api.notion.com
```

#### JSON-RPC Wire Format

```jsonc
// OM Bot  →  MCP Server
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "create_issue",
    "arguments": {
      "owner": "PREETCHAUHAN2005",
      "repo": "openclaw",
      "title": "Fix login timeout bug",
      "body": "Reported in #dev-team on Oct 5..."
    }
  }
}

// MCP Server  →  OM Bot
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [{ "type": "text", "text": "Created #42: https://github.com/..." }]
  }
}
```

#### Startup Handshake

```
For each configured MCP server:
  1. Spawn process
  2. Send  →  initialize
  3. Send  →  notifications/initialized
  4. Send  →  tools/list
  5. Store tool schemas for LLM injection
```

---

## Setup & Installation

### What You'll Need

```
✦ Node.js 18+
✦ A Slack workspace (with admin rights)
✦ OpenAI API key
✦ GitHub Personal Access Token
✦ Notion Internal Integration Token
✦ mem0.ai API key
```

---

### Step 1 — Clone the project

```bash
git clone https://github.com/yourusername/om-bot.git
cd om-bot
npm install
```

---

### Step 2 — Create your Slack App

1. Visit [api.slack.com/apps](https://api.slack.com/apps) → **Create New App → From scratch**
2. Under **Settings → Socket Mode**, enable Socket Mode
3. Under **OAuth & Permissions**, add these **Bot Token Scopes**:

```
app_mentions:read     channels:history     channels:read
chat:write            im:history           im:read
im:write              reactions:read       reactions:write
reminders:read        reminders:write      users:read
```

4. Add **User Token Scopes**: `reminders:read` · `reminders:write`
5. Install to workspace and save:
   - **Bot Token** — starts with `xoxb-`
   - **App Token** — starts with `xapp-`
   - **User Token** — starts with `xoxp-`

---

### Step 3 — Gather API credentials

**OpenAI** → [platform.openai.com](https://platform.openai.com) → Create API key

**GitHub** → [github.com/settings/tokens](https://github.com/settings/tokens) → New classic token with `repo` + `issues` scopes

**Notion** → [notion.so/my-integrations](https://www.notion.so/my-integrations) → New integration → Copy token → Share target pages with the integration

**mem0** → [app.mem0.ai](https://app.mem0.ai) → Sign up → Copy API key

---

### Step 4 — Configure environment

```bash
cp .env.example .env
```

```env
# ── Slack ───────────────────────────────────────────────────────
SLACK_BOT_TOKEN=xoxb-your-bot-token
SLACK_APP_TOKEN=xapp-your-app-token
SLACK_USER_TOKEN=xoxp-your-user-token       # optional, needed for reminders

# ── AI ──────────────────────────────────────────────────────────
OPENAI_API_KEY=sk-your-openai-key
DEFAULT_MODEL=gpt-4o

# ── Memory ──────────────────────────────────────────────────────
MEM0_API_KEY=m0-your-mem0-key
MEMORY_ENABLED=true

# ── MCP Integrations ────────────────────────────────────────────
GITHUB_PERSONAL_ACCESS_TOKEN=ghp_your-token
NOTION_API_TOKEN=secret_your-token

# ── RAG ─────────────────────────────────────────────────────────
RAG_ENABLED=true
RAG_INDEX_INTERVAL_MINUTES=60
```

---

### Step 5 — Run

```bash
npm run dev      # development (hot reload)
npm run build    # compile TypeScript
npm start        # production
```

**Healthy startup output:**
```
✅  Database initialized
✅  Vector store ready  ·  254 documents indexed
✅  Background indexer started  ·  interval: 60min
✅  Memory system connected  (mem0 cloud)
✅  MCP initialized  ·  servers: github, notion
✅  Task scheduler online
✅  Slack socket connected

  OM Bot is live.

  RAG (Semantic Search)    ✅  enabled
  Long-Term Memory         ✅  enabled
  MCP Tools (GitHub)       ✅  26 tools
  MCP Tools (Notion)       ✅  21 tools
  Slack Tools              ✅  12 tools
  AI Model                 ✅  gpt-4o

  Press Ctrl+C to stop.
```

---

### Optional — Custom MCP config

Create `mcp-config.json` to override defaults or add new servers:

```json
{
  "servers": [
    {
      "name": "github",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "$GITHUB_PERSONAL_ACCESS_TOKEN" }
    },
    {
      "name": "notion",
      "command": "npx",
      "args": ["-y", "@notionhq/notion-mcp-server"],
      "env": {
        "OPENAPI_MCP_HEADERS": "{\"Authorization\": \"Bearer $NOTION_API_TOKEN\", \"Notion-Version\": \"2022-06-28\"}"
      }
    }
  ]
}
```

---

## ⚙️ Environment Reference

| Variable | Required | Default | Notes |
|:---|:---:|:---:|:---|
| `SLACK_BOT_TOKEN` | ✅ | — | `xoxb-` prefix |
| `SLACK_APP_TOKEN` | ✅ | — | `xapp-` prefix, Socket Mode |
| `SLACK_USER_TOKEN` | ❌ | — | `xoxp-` prefix, needed for reminders |
| `OPENAI_API_KEY` | ✅ | — | Used for LLM + embeddings |
| `DEFAULT_MODEL` | ❌ | `gpt-4o` | Any OpenAI chat model |
| `MEM0_API_KEY` | ❌ | — | Disables memory if absent |
| `MEMORY_ENABLED` | ❌ | `true` | Hard toggle |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | ❌ | — | Disables GitHub MCP if absent |
| `NOTION_API_TOKEN` | ❌ | — | Disables Notion MCP if absent |
| `RAG_ENABLED` | ❌ | `true` | Hard toggle |
| `RAG_INDEX_INTERVAL_MINUTES` | ❌ | `60` | Background re-indexing cadence |
| `LOG_LEVEL` | ❌ | `info` | `debug` for verbose mode |

---

##  See It In Action

#### Pulling from Slack history
```
You  →  "What did the team say about the API timeout issue?"

OM   →  Based on indexed Slack messages:

        [Oct 15] @john  "The API timeout is hardcoded at 30s — should make it configurable"
        [Oct 16] @jane  "Agreed. PR is incoming."
        [Oct 17] @john  "Merged and deployed to staging."

        Summary: The team flagged a hardcoded 30s timeout, agreed to make it
        configurable, and shipped a fix to staging within 48 hours.
```

#### Memory across sessions
```
Session 1 ──────────────────────────────────────────────────────────────────
You  →  "Remember: I prefer Python over JavaScript for all scripts."
OM   →  "Noted — I'll default to Python for any code I write for you."

Session 2 (new day) ────────────────────────────────────────────────────────
You  →  "Write a hello world script."
OM   →  "Here you go (Python, as you prefer):

         print('Hello, world!')"
```

#### GitHub via MCP
```
You  →  "List my GitHub repos and create an issue for the login bug."

OM   →  Found your repositories:
         · nano-kimi       · Mixture_of_Experts       · ML-Teach-by-Doing
         · ...and 8 more

        Created issue #42 in VizuaraAI/nano-kimi:
        "Login timeout bug"  →  https://github.com/VizuaraAI/nano-kimi/issues/42
```

#### Notion via MCP
```
You  →  "Search Notion for the Q4 roadmap."

OM   →  Found 3 matching pages:
         1. Q4 Product Roadmap  (edited 2 days ago)
         2. Engineering Roadmap 2024
         3. Roadmap Template

        Want me to pull the content of any of these?
```

#### The power combo
```
You  →  "My GitHub is VizuaraAI. Find all bug discussions in Slack
         and open GitHub issues for each one."

OM   →  Remembered your GitHub handle.

        Searched Slack — found 10 messages referencing bugs.

        Created 2 GitHub issues:
          · #42  Login timeout bug         (from Oct 5 #dev-team discussion)
          · #43  API response caching      (from Oct 12 #backend discussion)

        Both are open in VizuaraAI/nano-kimi.
```

---

##  Full Tool Catalog

### Slack Built-in Tools (12)

| Tool | What It Does |
|:---|:---|
| `search_knowledge_base` | Semantic search across all indexed Slack messages |
| `send_message` | Send a message to any channel or user |
| `get_channel_history` | Fetch recent messages from a channel |
| `schedule_message` | Schedule a one-off message |
| `schedule_recurring_message` | Set up a recurring message |
| `set_reminder` | Create a Slack reminder |
| `list_channels` | List all workspace channels |
| `list_users` | List all workspace members |
| `get_my_memories` | Show what OM Bot remembers about you |
| `remember_this` | Explicitly save a fact |
| `forget_about` | Delete specific memories |
| `forget_everything` | Full memory wipe for your user |

### GitHub via MCP (26 tools)

| Tool | What It Does |
|:---|:---|
| `github_search_repositories` | Search for repositories |
| `github_get_repository` | Get full repo details |
| `github_list_issues` | List issues (filterable) |
| `github_create_issue` | Open a new issue |
| `github_get_issue` | Read a specific issue |
| `github_update_issue` | Edit or close an issue |
| `github_list_pull_requests` | List PRs |
| `github_create_pull_request` | Open a new PR |
| `github_get_file_contents` | Read any file in a repo |
| `github_search_code` | Search code across repos |
| *+ 16 additional tools* | Commits, branches, users, gists... |

### Notion via MCP (21 tools)

| Tool | What It Does |
|:---|:---|
| `notion_search` | Full-text search across all pages |
| `notion_get_page` | Read a page's full content |
| `notion_create_page` | Create a new page |
| `notion_update_page` | Edit page content or properties |
| `notion_query_database` | Query a Notion database with filters |
| `notion_create_database` | Create a new database |
| *+ 15 additional tools* | Comments, blocks, users, properties... |

---

## 📁 Project Map

```
om-bot/
│
├── src/
│   ├── index.ts                   ← entry point
│   │
│   ├── config/
│   │   └── index.ts               ← env loading & validation
│   │
│   ├── channels/
│   │   └── slack.ts               ← all Slack event handlers
│   │
│   ├── agents/
│   │   └── agent.ts               ← core AI loop + tool orchestration
│   │
│   ├── memory/
│   │   └── database.ts            ← SQLite session store
│   │
│   ├── memory-ai/
│   │   ├── index.ts               ← exports
│   │   └── mem0-client.ts         ← mem0 API integration
│   │
│   ├── rag/
│   │   ├── index.ts               ← exports
│   │   ├── vectorstore.ts         ← ChromaDB interface
│   │   ├── embeddings.ts          ← OpenAI embedding calls
│   │   ├── indexer.ts             ← background channel indexer
│   │   └── retriever.ts           ← semantic search logic
│   │
│   ├── mcp/
│   │   ├── index.ts               ← exports
│   │   ├── client.ts              ← MCP server manager
│   │   ├── config.ts              ← server configuration loader
│   │   └── tool-converter.ts      ← MCP schema → OpenAI format
│   │
│   ├── tools/
│   │   ├── slack-actions.ts       ← Slack API wrappers
│   │   └── scheduler.ts           ← task/reminder scheduler
│   │
│   └── utils/
│       └── logger.ts              ← Winston logger
│
├── data/                          ← local ChromaDB files (gitignored)
├── docs/                          ← extended documentation
├── scripts/                       ← setup & manual indexing utilities
├── .env.example
├── mcp-config.example.json
├── tsconfig.json
└── package.json
```

---

##  Troubleshooting Playbook

#### MCP server not connecting

```bash
# Verify tokens are exported
echo $GITHUB_PERSONAL_ACCESS_TOKEN
echo $NOTION_API_TOKEN

# Test GitHub token directly
curl -H "Authorization: token $GITHUB_PERSONAL_ACCESS_TOKEN" \
     https://api.github.com/user
```

---

#### RAG returning 0 results

```
1. Check startup log for: "Vector store ready · N documents indexed"
   If N = 0, the bot hasn't indexed any channels yet.

2. Invite the bot to your channels:  /invite @om-bot

3. Restart to trigger immediate indexing.
```

---

#### Memory not persisting

```
Look for this warning in logs:
"Failed to initialize client: ReferenceError: window is not defined"

This is a known issue with the mem0 npm package — memory still functions
correctly via the REST API. No action needed.
```

---

#### Bot ignores messages in channels

```
✦ Always mention the bot:   @om-bot your message
✦ Confirm bot is a member:  /invite @om-bot
✦ Check ALLOWED_CHANNELS in your config
```

---

#### LLM not picking up a tool

The model decides when to use tools. Be explicit to guarantee usage:

```
Less reliable  →  "What repos do I have?"
More reliable  →  "Use GitHub to list my repositories."
Most reliable  →  "Call the GitHub tool and list all repos for VizuaraAI."
```

---

#### Enable debug mode

```env
LOG_LEVEL=debug
```

Key lines to watch for:

```
Total tools available: 59 (12 Slack + 47 MCP)   ← all tools loaded
Executing tool: search_knowledge_base            ← Slack tool call
Executing MCP tool: github/create_issue          ← MCP routing
Stored 1 memories for user U050Y4SNQF3           ← memory persisted
```

---

## 🤝 Contributing

Pull requests are welcome.

```bash
# Fork, then:
git checkout -b feature/your-idea
git commit -m "feat: describe your change"
git push origin feature/your-idea
# Open a PR
```

**For local development:**
```bash
npm install
npm run dev          # hot-reload server
npm run typecheck    # TypeScript validation
npm run lint         # ESLint
```

---

## 📄 License

MIT — see [LICENSE](LICENSE).

---

## 🙏 Built On

[Slack Bolt.js](https://slack.dev/bolt-js/) · [OpenAI](https://openai.com/) · [mem0](https://mem0.ai/) · [Model Context Protocol](https://modelcontextprotocol.io/) · [ChromaDB](https://www.trychroma.com/)

---

<div align="center">

*OM Bot — because your team's knowledge shouldn't live and die in a chat scroll.*

⭐ If this saves you time, a star goes a long way.

 Build with ❤️ by Preet Chauhan.

</div>
