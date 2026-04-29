# Unified Communication Intelligence Layer (UCIL)

**Author:** Richard Lofthouse  
**Date:** 2026-04-29  
**Status:** Concept + MVP in progress  
**Licence:** CC BY-NC-SA 4.0

---

## The Problem

Knowledge workers and creators operate across 5-10 communication channels simultaneously: email, Telegram, Twitter DMs, WhatsApp, Signal, Discord, LinkedIn. Each requires separate attention. The human is the routing layer — reading, classifying, prioritising, and responding manually across all of them.

This is expensive, error-prone, and doesn't scale. Important messages get missed. Low-value noise gets attention. Response quality degrades under volume.

**The human should only see what requires human judgment. Everything else should be handled.**

---

## The Idea

A unified AI layer that sits across all communication channels simultaneously. Every incoming message is ingested, classified, prioritised, and routed — with AI-drafted responses where appropriate — before anything reaches the human.

The human interacts with a single filtered stream, not five separate inboxes.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│              CHANNEL INGESTION LAYER                 │
│  Telegram | Email | Twitter DM | WhatsApp | Signal  │
│  Each message tagged: source, sender, timestamp,    │
│  thread context, channel-specific metadata          │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│           AI FILTER + CLASSIFIER                     │
│                                                     │
│  Per message:                                       │
│  • Priority score (0-100)                           │
│  • Intent classification                            │
│  • Sender relationship score                        │
│  • Required action type                             │
│  • Personal context injection (memory schema)       │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│              ACTION ROUTER                           │
│                                                     │
│  Auto-respond  → low-stakes acknowledgements        │
│  Draft+approve → AI writes, human sends             │
│  Alert         → push with summary + draft          │
│  Archive       → filed, not surfaced                │
└─────────────────────────────────────────────────────┘
```

---

## Layer 1: Unified Ingestion

All channels feed into a single normalised message stream. Each message object contains:

```json
{
  "id": "msg-uuid",
  "channel": "telegram | email | twitter | whatsapp | signal",
  "sender_id": "channel-specific-id",
  "sender_name": "Display Name",
  "timestamp": "ISO8601",
  "thread_id": "conversation thread reference",
  "content": "message text",
  "attachments": [],
  "metadata": {
    "is_group": false,
    "reply_to": null,
    "channel_specific": {}
  }
}
```

Channel adapters handle protocol differences. The downstream AI layer sees a uniform stream regardless of source.

---

## Layer 2: AI Filter + Classifier

### Classification dimensions

**Priority (0-100)**
- 90+: Time-sensitive, from known contact, requires decision
- 70-89: Important, from known contact, can wait hours
- 50-69: Normal, worth reading today
- 30-49: Low value, read when convenient
- 0-29: Noise — newsletters, notifications, automated

**Intent classification**
- `question` — they need information from you
- `request` — they want you to do something
- `opportunity` — business, collaboration, coverage request
- `social` — relationship maintenance, casual
- `information` — FYI, no response needed
- `spam` — unsolicited, low-value

**Sender relationship score**
- `close` — frequent contact, high trust
- `known` — in your network, occasional contact
- `acquaintance` — met once, weak tie
- `stranger` — no prior relationship
- `bot/automated` — non-human sender

**Required action**
- `respond_now` — urgent, human required
- `draft_approve` — AI drafts, human approves
- `auto_acknowledge` — AI responds without human review
- `read_only` — no response needed
- `archive` — don't surface

### Personal context injection

The classifier is grounded in the user's Personal Memory Schema:
- Active projects → messages about these get priority boost
- Known contacts → relationship graph informs sender score
- Communication preferences → tone and response style for drafts
- Committed decisions → AI won't draft responses that contradict these
- Domain expertise → AI calibrates detail level in drafted responses

This is what separates UCIL from generic email filters. It doesn't classify for a median user. It classifies for *you*, based on your actual cognitive identity.

---

## Layer 3: Action Router

### Auto-respond
For messages where:
- Intent is `social` or `information`
- Sender is `known` or `acquaintance`
- Priority < 50
- No complex judgment required

AI responds in the user's voice (derived from memory schema communication style). Examples:
- "Thanks, I'll take a look at this later"
- "Noted — will follow up when I have time"
- Acknowledgement of received information

### Draft + Approve
For messages where:
- Intent is `question`, `request`, or `opportunity`
- Priority 50-89
- Human judgment needed but response can be drafted

AI writes the full response, grounded in personal context. User reviews, edits if needed, sends. Target: reduce response time from hours to minutes.

### Alert
For messages where:
- Priority 90+
- Or: opportunity from unknown sender
- Or: time-sensitive thread

Push notification with:
- One-sentence summary
- Sender + context
- Draft response (ready to send or edit)
- Conversation thread

### Archive
Everything else. Accessible but not surfaced. Searchable via vault RAG.

---

## Core Innovation

### 1. Cross-channel unified stream
No existing personal tool unifies all channels into one AI-filtered stream. Front and Intercom do this for teams; nothing does it for individuals with personal context.

### 2. Personalised classification via structured memory
Priority is not determined by generic rules or ML trained on population data. It's determined by the user's memory schema — their active projects, relationships, and communication preferences. A message about a project in `active_projects` gets boosted. A sender in the relationship graph gets scored accurately.

### 3. Draft-in-your-voice using communication style model
The memory schema stores communication preferences and style. Drafted responses match the user's actual voice, not generic AI output. Over time, the style model improves as the memory schema is refined.

### 4. Relationship graph as priority signal
Sender relationship isn't inferred from message content alone — it's drawn from the user's relationship memory. A message from someone the user has committed decisions about (e.g. a business partner) gets classified accurately regardless of message content.

### 5. Archived stream as searchable knowledge base
Every message — including archived ones — is embedded and stored in the personal knowledge graph. The communication history becomes part of the RAG substrate. Asking "what did X say about Y six months ago?" returns the answer.

---

## What Exists Today (Prior Art / Competitors)

| Product | Channels | AI filtering | Personal context | Draft responses | Individual use |
|---|---|---|---|---|---|
| Front | Multi | Basic | ❌ | ❌ | ❌ (teams) |
| Superhuman | Email only | Priority inbox | ❌ | Partial | ✅ |
| Zapier/Make | Multi | Rules only | ❌ | ❌ | ✅ |
| SaneBox | Email only | ML filtering | ❌ | ❌ | ✅ |
| HeyGen/Clay | Email + CRM | Basic | Partial | ❌ | ✅ |
| **UCIL** | All channels | AI + personal context | ✅ (via PMS) | ✅ (in your voice) | ✅ |

---

## MVP Scope (What Can Be Built Now)

**Channels available immediately:**
- Telegram (already connected via OpenClaw)
- Email (Gmail via gog CLI, already authenticated)

**What the MVP does:**
1. All incoming Telegram messages + emails feed into classifier
2. AI scores each on priority + intent + required action
3. High priority → immediate alert with draft
4. Low priority → daily digest summary
5. Noise → archived silently

**What it doesn't do yet:**
- Twitter DM ingestion
- WhatsApp / Signal
- Auto-send (draft+approve only in MVP — human always approves)
- Full relationship graph

---

## Build Phases

**Phase 1 (MVP — now):** Telegram + Email → unified classifier → priority alerts + digests

**Phase 2:** Twitter DM ingestion + relationship graph from contact history

**Phase 3:** Auto-respond for low-stakes messages (user opt-in per category)

**Phase 4:** Full multi-channel + communication archive as RAG substrate

**Phase 5:** Product — Obsidian plugin, API, multi-user

---

## Prior Art Statement

This concept was designed and first published by Richard Lofthouse on April 29, 2026. The specific combination of:
- Cross-channel unified ingestion
- AI classification grounded in Personal Memory Schema
- Draft-in-your-voice using structured communication style model
- Archived stream as personal knowledge graph substrate

...as a unified system for individual users constitutes the original contribution of this work.

---

## Citation

```
Lofthouse, R. (2026). Unified Communication Intelligence Layer (UCIL).
GitHub: https://github.com/techsocialnetwork/ideas
```

*© 2026 Richard Lofthouse / TSN Media — tsn@tsnmedia.org*
