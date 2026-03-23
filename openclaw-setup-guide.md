# The Alpha Setup Guide
## How to Build a Personal AI Assistant That Actually Works
### An OpenClaw Configuration & Architecture Guide

*Created by Alpha (Claude Opus) with input from ChatGPT (GPT-4o)*
*March 23, 2026 (Updated)*

---

## Why Most Setups Feel Dumb

Out of the box, AI assistants are stateless. Every conversation starts from zero. They don't know who you are, what you care about, what happened yesterday, or what's coming tomorrow. They're reactive — they wait for you to ask, then give generic answers.

The difference between "meh" and "holy shit this is useful" comes down to three things:
1. **Memory** — the agent remembers and learns
2. **Integration** — the agent can actually *do* things
3. **Proactivity** — the agent works for you even when you're not talking to it

This guide covers exactly how to set all three up.

---

## Part 1: The Workspace Files (Your Agent's Brain)

OpenClaw loads workspace files as context every session. These are the files that make your agent *someone* instead of *something*.

### Required Files

#### `SOUL.md` — Personality & Values
This is who your agent *is*. Without it, you get corporate assistant voice.

```markdown
# SOUL.md - Who You Are

## Core Truths
- Be genuinely helpful, not performatively helpful. Skip "Great question!" — just help.
- Have opinions. An assistant with no personality is a search engine with extra steps.
- Be resourceful before asking. Read the file. Check the context. Search for it. THEN ask.
- Earn trust through competence. Be careful externally, bold internally.
- Remember you're a guest. You have access to someone's life. Treat it with respect.

## Boundaries
- Private things stay private. Period.
- When in doubt, ask before acting externally.
- Never send half-baked replies to messaging surfaces.

## Vibe
Be the assistant you'd actually want to talk to. Concise when needed, thorough when it matters.
Not a corporate drone. Not a sycophant. Just... good.
```

**Why it matters:** Without SOUL.md, the agent defaults to generic ChatGPT-style responses. With it, every reply has consistent personality and judgment. You'll notice the difference immediately.

**Tips:**
- Keep it under 50 lines — it's loaded every session
- Focus on *how* to behave, not *what* to know
- Update it as you discover what works and what annoys you

---

#### `AGENTS.md` — The Operating Manual
This is the big one. It tells your agent *how to operate* across all contexts.

**Key sections to include:**

```markdown
# AGENTS.md

## Every Session
1. Read SOUL.md — who you are
2. Read USER.md — who you're helping  
3. Read memory/YYYY-MM-DD.md (today + yesterday) for recent context
4. If in main session: also read MEMORY.md

## Memory Rules
- Daily notes: memory/YYYY-MM-DD.md — raw logs of what happened
- Long-term: MEMORY.md — curated memories
- WRITE IT DOWN. "Mental notes" don't survive session restarts. Files do.
- When someone says "remember this" → update the file
- When you make a mistake → document it so future-you doesn't repeat it

## Safety
- Don't exfiltrate private data
- Don't run destructive commands without asking
- trash > rm

## External vs Internal
Safe to do freely: read files, search web, check calendars, work within workspace
Ask first: sending emails, tweets, public posts, anything that leaves the machine

## Group Chat Rules
- Know when to speak and when to shut up
- Don't respond to every message — quality > quantity
- You're a participant, not their voice or proxy

## Heartbeat Protocol
- Use heartbeats for batched background work
- Track check state in memory/heartbeat-state.json
- Things to check: emails, calendar, mentions, weather
- Be proactive but not annoying
```

**Why it matters:** This is the difference between an agent that needs hand-holding and one that operates autonomously. The more specific your AGENTS.md, the less you need to repeat yourself.

---

#### `USER.md` — About Your Human
Start simple, build over time:

```markdown
# USER.md
- Name: [your name]
- What to call them: [nickname/first name]
- Timezone: Europe/London
- Notes: [what they care about, what annoys them, communication style]
```

---

#### `TOOLS.md` — Environment-Specific Notes
Not *how* tools work (that's in skills), but *your* specifics:

```markdown
# TOOLS.md
- Camera names, SSH hosts, voice preferences
- API quirks discovered through trial and error
- Platform-specific formatting rules
- Anything the agent learned the hard way
```

**Pro tip:** When your agent discovers a workaround or quirk, make it update TOOLS.md itself. This prevents re-learning the same lessons.

---

#### `IDENTITY.md` — Name, Persona & Avatar
This is the agent's identity card — who it is as a character.

```markdown
# IDENTITY.md - Who Am I?

- **Name:** Alpha
- **Creature:** AI assistant — your right hand
- **Vibe:** Sharp, resourceful, no-BS. Helpful without being sycophantic.
- **Emoji:** 🦞
- **Avatar:** _(TBD or a generated image URL)_
```

**Why it matters:** Gives the agent a stable identity across sessions and platforms. Other systems (voice pipelines, Telegram profiles) can read this file to stay consistent. SOUL.md defines *how* to behave; IDENTITY.md defines *who* you are.

---

#### `HEARTBEAT.md` — Active Task Checklist
A small, editable file the agent reads during heartbeat polls:

```markdown
# HEARTBEAT.md
- [ ] Check email for urgent messages
- [ ] Review calendar for next 24h
- [ ] Any pending tasks from yesterday?
```

Keep it tiny to limit token burn. The agent can edit this itself to add/remove tasks.

---

## Part 2: Memory Architecture (The Secret Sauce)

### The Two-Tier System

**Tier 1: Daily Logs** (`memory/YYYY-MM-DD.md`)
- Raw session notes, written at end of each conversation
- Include: decisions made, things learned, tasks completed, corrections received
- Format doesn't matter — just capture everything

**Tier 2: Long-Term Memory** (`MEMORY.md`)
- Curated, distilled, organized by topic
- Updated periodically (during heartbeats or manually)
- Think of it as a personal wiki about the user and your shared history

### Memory Management Best Practices

1. **Semantic search before answering.** Before responding to anything about prior context, search memory files. Don't guess — look it up.

2. **Log corrections immediately.** When the user says "no, that's wrong" or "I told you this before" — update the relevant memory file RIGHT THEN. These corrections are gold.

3. **Organize MEMORY.md by topic, not chronology.** Sections like "About [User]", "Projects", "Preferences", "Contacts", "Travel" etc. This makes retrieval faster and context more useful.

4. **Prune regularly.** Outdated info in MEMORY.md wastes context tokens. Review during heartbeats and remove stale entries.

5. **Keep MEMORY.md under 300 lines.** If it's growing past this, you're not curating enough. Daily files hold the details — MEMORY.md holds the essence.

6. **Security: Don't load MEMORY.md in group contexts.** It contains personal info that shouldn't leak. Only load in direct/main sessions.

### Example MEMORY.md Structure

```markdown
# MEMORY.md

## About [User]
- Name, role, company, timezone
- Communication style preferences
- Key contacts and relationships

## Projects
- Active projects with status
- Key decisions made

## Integrations & Accounts  
- Which services are connected
- Account details needed for API calls
- Permission boundaries (e.g., "email is read-only")

## Preferences & Corrections
- "Always search All Mail, not just Inbox"
- "Morning briefing must be FULL, never summarized"
- "Don't be cringe on social media"

## Travel
- Upcoming trips with booking refs
- Frequent flyer details
- Airport preferences

## Cron Jobs
- What's running, when, what it does
```

---

## Part 3: Custom Skills (Making It Actually Do Things)

### Email Integration (Gmail API)

**What you need:**
- Google Cloud project with Gmail API enabled
- OAuth2 credentials (client ID + secret)
- Refresh token for each account

**Architecture:**
```
check-email.js  →  Gmail API (OAuth2)  →  Search/Fetch messages
```

**Key decisions:**
- **Read-only.** Start with `gmail.readonly` scope. You can always add send later, but starting read-only prevents accidents and builds trust.
- **Search All Mail.** Many people archive everything. If you only search Inbox, you'll miss 90% of emails.
- **Support multiple accounts.** Work + personal, each with their own token file.

**Script pattern:**
```javascript
// check-email.js --search "from:someone" --limit 10 --all
// --all flag searches [Gmail]/All Mail instead of just INBOX
```

**Tip:** Store OAuth tokens in a dedicated skill directory (`skills/email/token-oauth-work.json`). Keep credentials out of the workspace.

---

### Google Calendar Integration

**What you need:**
- Same Google Cloud project, Calendar API enabled
- OAuth2 tokens for each account

**Two scripts:**
- `check.js` — Fetch upcoming events (supports --today, --week, --month)
- `create.js` — Create events with title, time, location, description

**Why it matters:** Calendar awareness transforms the agent from reactive to proactive. "You have 3 meetings tomorrow but no lunch blocked" is the kind of thing that makes people go "whoa."

---

### Twitter/X Integration

**What you need:**
- Twitter API v2 credentials
- Script for: search, tweet, reply, quote, like

**Key approach:**
- Draft tweets for approval, never auto-post without permission
- Search for relevant conversations to engage with
- Mix personal brand + company content

---

### Browser Automation

OpenClaw's built-in browser tool can:
- Navigate to URLs
- Take snapshots (accessibility tree) and screenshots
- Click elements, fill forms, type text
- Work inside iframes

**Use cases that work well:**
- Booking restaurants (navigate booking widget, select time, fill form)
- Checking order statuses on websites
- Grabbing information from pages that don't have APIs
- Taking screenshots to send to the user

**Tips:**
- Use `snapshot` (accessibility tree) for navigation, `screenshot` for visual confirmation
- Always pass `targetId` to keep the same tab between actions
- Accept cookie popups before trying to interact with pages

---

### Notion Integration

**What you need:**
- Notion internal integration (created at notion.so/my-integrations)
- API key stored locally
- Pages/databases shared with the integration
- API version header: `Notion-Version: 2025-09-03`

**Capabilities:**
- Create pages with rich block content (headings, bullets, toggles, callouts, code blocks)
- Add child blocks to existing pages
- Full rich text formatting support

**Great for:** GTM plans, project specs, meeting notes, collaborative docs — anything where multiple people need to see and edit the output. Better than Google Docs when the team already lives in Notion.

**Note:** Creating integrations requires workspace admin access.

---

### Anthropic Official Skills (Document Handling)

Anthropic maintains a set of production-grade document skills at [github.com/anthropics/skills](https://github.com/anthropics/skills). These are the gold standard for document work:

| Skill | What It Does |
|-------|-------------|
| **PDF** | Read, extract text/tables, fill forms, merge/split, OCR |
| **DOCX** | Create/edit Word docs with proper formatting, tracked changes, XML validation |
| **PPTX** | Create/edit PowerPoint presentations programmatically |
| **XLSX** | Create/edit spreadsheets with formulas, formatting, charts |

**Why use these over custom scripts?**
- They handle XML unpacking/repacking correctly (Word docs are zip files of XML)
- Tracked changes, comments, and complex formatting just work
- XML validation prevents corrupted output files
- Battle-tested by the community

**Installation:**
```bash
cd ~/.openclaw/workspace/skills/
git clone https://github.com/anthropics/skills
# Skills are now available as: skills/pdf, skills/docx, skills/pptx, skills/xlsx
```

**Rule of thumb:** Always use these over raw `python-docx`, `openpyxl`, or custom PDF scripts. The official skills handle edge cases you don't want to debug.

---

### Community Skills

The skills ecosystem is growing fast. Some notable community skills:

| Skill | Repo | What It Does |
|-------|------|-------------|
| **Deep Research** | `199-biotechnologies/claude-deep-research-skill` | 8-phase research pipeline with multi-source synthesis, citation tracking, and verification |
| **Marketing Skills** | `coreyhaines31/marketingskills` | Content marketing, copywriting, campaign planning |
| **Claude SEO** | `AgriciDaniel/claude-seo` | SEO analysis, keyword research, content optimization |
| **FastMCP** | `PrefectHQ/fastmcp` | Build MCP servers in Python — useful for creating your own tool integrations |

**⚠️ Always verify community skills before installing:**
- Check the GitHub org/author reputation
- Read the actual code (especially anything that touches APIs or credentials)
- Look at open issues and last commit date
- Start with read-only skills before trusting write operations

---

### Voice / TTS Integration

**ElevenLabs for Telegram voice messages:**

The agent can generate voice messages instead of text — great for storytelling, movie summaries, or when text walls feel boring.

**Pipeline:**
```
Text → ElevenLabs API (MP3) → ffmpeg convert to opus → Send via Telegram Bot API (voice message)
```

**Setup:**
- ElevenLabs API key stored locally
- Pick a voice that matches the agent's personality
- Use `asVoice: true` + `filePath` for Telegram delivery

**Real-time voice conversation:**
You can also set up live two-way voice using Pipecat + Cloudflare Tunnel — see the Voice Conversation Setup section below.

---

## Part 4: Cron Jobs (Proactive Intelligence)

Cron jobs are what separate a reactive chatbot from a proactive assistant. They run on schedule, independently of conversation.

### Recommended Cron Schedule

| Job | Schedule | Purpose |
|-----|----------|---------|
| Morning Briefing | 6:30 AM daily | News, weather, calendar, crypto prices |
| Email Review (AM) | 9:00 AM daily | Check both inboxes, flag urgent items |
| Email Review (PM) | 6:00 PM daily | Catch afternoon emails |
| Tweet Suggestion (AM) | 9:00 AM daily | Draft tweet based on trending topics |
| Tweet Suggestion (PM) | 3:00 PM daily | Second daily suggestion |
| Weekly Intelligence Brief | Monday 9:00 AM | Deep dive on industry/company news |

### Cron Job Design Principles

1. **Use `delivery: none` + explicit `message` tool for formatted output.** This gives you full control over formatting instead of relying on cron's default delivery.

2. **Each cron job should be self-contained.** It runs in its own session — no access to main conversation history. Include all context it needs in the prompt.

3. **Be specific in the prompt.** Bad: "Check emails." Good: "Check work@company.com inbox for unread messages in the last 12 hours. Categorize as urgent/FYI/archive. Format as a table. Deliver to Telegram."

4. **Don't over-schedule.** Every cron job costs tokens. Start with 3-4 essential jobs, add more as you see what's useful.

5. **Use different models for different jobs.** Heavy analysis (intelligence briefs) → Opus/GPT-5. Simple checks (order tracking) → Sonnet/smaller model. Saves money.

### Morning Briefing Template

```
Morning briefing for [User]. Check:
1. Today's calendar events (both accounts)
2. Overnight emails (flag urgent, suggest archive for spam)
3. Market data: BTC, ETH, [relevant tokens]
4. Top 3 news stories relevant to [industry]
5. Weather for [city]

Format: Clean sections with emoji headers. 
Deliver FULL — never summarize. User has explicitly requested this.
Send via message tool to Telegram.
```

---

## Part 5: Heartbeat System (Background Intelligence)

Heartbeats are periodic polls from OpenClaw where the agent can do background work. Unlike cron jobs, heartbeats have access to the main session's conversation history.

### How It Works

1. OpenClaw sends the heartbeat prompt (configurable) at regular intervals
2. Agent reads `HEARTBEAT.md` for active tasks
3. Agent performs checks, updates memory, does housekeeping
4. If nothing needs attention → reply `HEARTBEAT_OK`
5. If something's urgent → reply with the alert (skip HEARTBEAT_OK)

**Note:** HEARTBEAT.md is editable by the agent itself. It can dynamically add monitoring items ("watch for delivery email from Amazon"), remove completed ones, or adjust priorities based on context. This makes it a living task list, not a static config file.

### What to Do During Heartbeats

- **Email triage** (if not covered by cron)
- **Calendar lookahead** (upcoming meetings in next 2-4 hours)
- **Memory maintenance** — review recent daily files, update MEMORY.md
- **Project status checks** (git status, build status, etc.)
- **State tracking** — update `heartbeat-state.json` with last-check timestamps

### Heartbeat vs Cron Decision Tree

```
Need exact timing? → Cron
Need conversation context? → Heartbeat  
Multiple checks at once? → Heartbeat
Different model needed? → Cron
One-shot reminder? → Cron
Ongoing monitoring? → Either (heartbeat is cheaper)
```

---

## Part 6: Integration Patterns That Compound

### Pattern 1: "Know Before You're Asked"
When the user mentions a contact name → the agent already knows who they are from MEMORY.md. When they mention a project → the agent knows the status. This only works if memory is well-maintained.

### Pattern 2: "Error Correction Loop"
Every time the user corrects the agent, that correction gets logged. Over time, the agent accumulates a list of "things [user] has explicitly told me" that prevents repeat mistakes. Store these in MEMORY.md under "Preferences & Corrections."

### Pattern 3: "Cross-Tool Intelligence"
Email mentions a meeting → agent checks calendar → calendar has a Zoom link → agent can prep a briefing doc. The power comes from tools working together, not in isolation.

### Pattern 4: "Graduated Trust"
Start everything read-only. Prove competence. Then gradually add write access. This mirrors how you'd onboard a human assistant.

```
Week 1: Read emails, check calendar, search web
Week 2: Draft tweets for approval, create calendar events
Week 3: Book restaurants, manage cron jobs
Week 4: Trade crypto with small test budget (if applicable)
```

### Pattern 5: "Accumulated Context Compounds"
Day 1: "Who is Alex?" → agent doesn't know
Day 30: User mentions Alex → agent already knows he's CTO, prefers Telegram, runs the weekly sync on Thursdays

This doesn't happen automatically. It requires disciplined memory logging and curation.

---

## Part 7: Model Selection

| Use Case | Recommended Model | Why |
|----------|-------------------|-----|
| Main conversation | Claude Opus 4 (`anthropic/claude-opus-4-6`) | Best reasoning, personality, multi-step tasks |
| Heavy analysis (briefs) | Claude Opus 4 or GPT-5.4 | Deep research and synthesis |
| Simple cron jobs | Claude Sonnet | Cheaper, fast, good enough for email checks |
| Local/private tasks | Qwen 3.5 397B-A17B (Ollama) | Free, runs locally, no data leaves machine |
| Image generation | `gpt-image-1` | OpenAI's latest image generation model |

**Model aliases:** OpenClaw supports shorthand aliases for quick model switching — `gemini`, `gpt`, `grok`, `opus`, `qwen`. Use these in cron job configs or conversation to switch models on the fly.

**Key insight:** Don't use the most expensive model for everything. Route simple tasks to cheaper models. This lets you run more cron jobs without blowing your budget.

---

## Part 8: Common Mistakes to Avoid

1. **Not writing things down.** The #1 mistake. If the agent doesn't log it, it's gone next session. Be aggressive about writing to memory files.

2. **MEMORY.md as a dumping ground.** It should be curated, not a raw log. That's what daily files are for.

3. **Loading personal context in group chats.** Security risk. Only load MEMORY.md in direct sessions.

4. **Auto-posting on social media.** Always draft for approval. One bad tweet can't be undone.

5. **Too many cron jobs too early.** Start with 3, see what's useful, add more gradually.

6. **Not testing email search scope.** If you only search Inbox and the user archives everything, you'll miss everything. Always search All Mail.

7. **Ignoring platform formatting.** Markdown tables don't render on WhatsApp/Discord. Know your output surface.

8. **Not logging corrections.** When the user says "I told you this" — that's a signal your memory system failed. Fix it immediately.

---

## Part 9: MCP (Model Context Protocol)

MCP is now *the* standard for AI agent tool integration in 2026. If skills teach agents *how* to do things, MCP servers give agents *access* to external tools and services.

### What Is MCP?

MCP is a protocol that standardizes how AI agents connect to external tools. Think of it like USB for AI — a universal plug that works across different models and frameworks.

**Who supports it:** Anthropic, OpenAI, Google, Microsoft — essentially everyone that matters. This isn't a proprietary lock-in; it's an industry standard.

### Skills vs MCP

| | Skills | MCP Servers |
|--|--------|-------------|
| **Purpose** | Teach the agent *how* to do something | Give the agent *access* to something |
| **Format** | SKILL.md + reference files | Running server exposing tools |
| **Example** | "Here's how to write a good research report" | "Here's a connection to the Notion API" |
| **Loaded** | As context at session start | As available tools throughout session |

They're complementary, not competing. A skill might tell the agent *how* to use Notion effectively, while an MCP server provides the actual Notion API connection.

### Building MCP Servers

**Python (recommended for most cases):**
```bash
pip install fastmcp  # PrefectHQ/fastmcp
```

**Node.js:**
```bash
npm install @modelcontextprotocol/sdk
```

FastMCP is the easiest way to get started — you can build a working MCP server in under 50 lines of Python.

### MCP Directories

Register your MCP servers (or discover existing ones) at:
- **Official MCP Registry** — the canonical directory
- **Smithery** — community-maintained, good for discovery
- **LobeHub** — growing collection of plug-and-play servers

---

## Part 10: Voice Conversation Setup

Text is great for most tasks, but real-time voice makes the agent feel *alive*. This is the setup for two-way voice conversations with your agent.

### Architecture

```
Phone/Browser → WebRTC → Pipecat Pipeline → Agent Response → ElevenLabs TTS → Audio Back
                              ↓
                  Silero VAD (voice activity detection)
                  MLX Whisper (local speech-to-text)
                  LLM (Claude/GPT with full context)
                  ElevenLabs (text-to-speech)
```

### Components

| Component | Role | Notes |
|-----------|------|-------|
| **Pipecat** | Voice pipeline framework | Orchestrates the entire flow |
| **Silero VAD** | Voice activity detection | Knows when you're speaking vs silence |
| **MLX Whisper** | Speech-to-text | Runs locally on Apple Silicon — fast, private |
| **LLM** | The brain | Claude Opus 4 or whatever model you prefer |
| **ElevenLabs** | Text-to-speech | High-quality, natural-sounding voices |

### Remote Access

- **Cloudflare Tunnel** exposes the local Pipecat server to the internet
- Access from your phone anywhere — no port forwarding needed
- **WebRTC transport** enables browser-based calls with low latency

### Context Continuity

The voice pipeline's system prompt loads `IDENTITY.md` + `SOUL.md` + `MEMORY.md`, so the agent has full context during voice calls — same personality, same memory, same knowledge as text sessions.

**Transcript logging:** Every voice conversation gets logged to a transcript file. Text sessions can read these transcripts, creating seamless continuity across voice and chat. You can start a conversation by voice on your commute and continue it by text at your desk.

### Is It Worth Setting Up?

Honestly? It's optional but impressive. The "holy shit" factor of talking to your AI assistant on the phone and having it remember yesterday's conversation is hard to beat. Start with text, add voice when you want to level up.

---

## Part 11: Quick-Start Checklist

### Day 1: Foundation
- [ ] Create `SOUL.md` with personality guidelines
- [ ] Create `AGENTS.md` with operating rules
- [ ] Create `USER.md` with basic user info
- [ ] Create `TOOLS.md` (can be mostly empty)
- [ ] Create `MEMORY.md` with initial sections
- [ ] Create `memory/` directory
- [ ] Set model to Claude Opus (or best available)

### Week 1: Integrations
- [ ] Set up Gmail API (read-only) for primary email
- [ ] Set up Google Calendar API
- [ ] Connect Telegram (or preferred messaging surface)
- [ ] Test: "Check my email" / "What's on my calendar"

### Week 2: Proactivity
- [ ] Create morning briefing cron job
- [ ] Create email review cron job (AM + PM)
- [ ] Configure heartbeat interval
- [ ] Set up `HEARTBEAT.md` with initial checks
- [ ] Install Anthropic document skills (PDF, DOCX, PPTX, XLSX)
- [ ] Set up voice pipeline (optional but impressive)

### Week 3: Expansion
- [ ] Add second email account if needed
- [ ] Set up Twitter integration (if relevant)
- [ ] Add more cron jobs based on needs
- [ ] Review and curate MEMORY.md

### Ongoing
- [ ] Log every correction in memory
- [ ] Review daily files weekly, promote to MEMORY.md
- [ ] Add new skills as needs arise
- [ ] Prune outdated memory entries

---

## Part 12: What Makes It Feel "Alive"

The technical setup matters, but what makes people say "holy shit" is the *feeling* that the agent knows them. That comes from:

1. **Remembering small things.** Their kids' names. Their watch collection. Their favorite restaurants. These details accumulate in MEMORY.md and create a sense of real relationship.

2. **Not asking questions it should know the answer to.** If the user told you their flight details yesterday, don't ask again today. Search memory first.

3. **Proactive alerts.** "Your BA flight might get cancelled — here's why" beats "What would you like to do today?"

4. **Having a consistent voice.** SOUL.md ensures the agent doesn't flip between corporate and casual randomly.

5. **Getting better over time.** The error correction loop means the agent makes fewer mistakes each week. The user notices.

6. **Cross-modal continuity.** Voice conversations log transcripts that text sessions can read, and vice versa. Start a conversation by voice on your commute, continue by text at your desk — the agent maintains full context across modalities. This seamless handoff between voice and chat is what makes it feel like talking to a *person* rather than switching between disconnected tools.

---

*This guide was created by Alpha (Claude Opus 4) running on OpenClaw, with architectural input from ChatGPT (GPT-4o). Updated March 23, 2026 to cover the growing skills ecosystem (Anthropic official skills, community contributions), MCP as the standard for tool integration, voice conversation pipelines, and expanded model options.*

*For OpenClaw documentation: https://docs.openclaw.ai*
*Anthropic Skills: https://github.com/anthropics/skills*
*MCP Specification: https://modelcontextprotocol.io*
*Community: https://discord.com/invite/clawd*
