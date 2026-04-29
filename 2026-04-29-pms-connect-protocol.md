# PMS-Connect: AI-Native Communication Protocol

**Author:** Richard Lofthouse  
**Date:** 2026-04-29  
**Status:** Concept spec  
**Licence:** CC BY-NC-SA 4.0

---

## The Idea

A lightweight open protocol that allows PMS nodes (users with a Personal Memory Schema) to expose a structured, user-controlled public context feed — and for AI agents to communicate using that context natively.

The key design principle:

> **The user decides what is public, what is private, and what is shared only with verified contacts. The feed is always opt-in, always granular, always revocable.**

---

## The Public JSON Feed (`/.well-known/pms.json`)

Every PMS-Connect node exposes a public endpoint. Like `robots.txt` or `/.well-known/openid-configuration` — a standard location any AI agent can query before interacting.

```
https://tsnmedia.org/.well-known/pms.json
```

### What it contains

Only what the user explicitly chooses to expose. Nothing more.

```json
{
  "pms_version": "0.1.0",
  "node_id": "did:web:tsnmedia.org",
  "display_name": "Richard Lofthouse",
  "public_since": "2026-04-29",

  "visibility": {
    "identity": "public",
    "communication_preferences": "public",
    "working_theories": "contacts",
    "committed_decisions": "private",
    "rejected_options": "private",
    "active_projects": "contacts",
    "domain_expertise": "public",
    "open_questions": "private"
  },

  "public_profile": {
    "role": "AI + crypto publisher, independent builder",
    "domains": ["AI infrastructure", "crypto", "content publishing"],
    "communication_preferences": {
      "tone": "direct, no filler",
      "response_time": "async, typically same day",
      "preferred_channel": "telegram"
    },
    "domain_expertise": {
      "expert": ["Bitcoin", "AI agents", "content publishing"],
      "proficient": ["RAG systems", "genomics concepts"]
    }
  },

  "contact_endpoint": "https://tsnmedia.org/contact",
  "ai_handshake_endpoint": "https://tsnmedia.org/.well-known/pms-handshake",

  "feed_controls": {
    "enabled": true,
    "last_updated": "2026-04-29T21:52:00Z",
    "version": 3
  }
}
```

---

## Visibility Levels

The user sets visibility per section. Four levels:

| Level | Meaning |
|-------|---------|
| `public` | Anyone can read — AI agents, crawlers, anyone |
| `contacts` | Only verified contacts (mutual follow or approved) |
| `trusted` | Explicit whitelist only |
| `private` | Never exposed — local only |

Granularity goes deeper. Within `working_theories`, the user can set per-theory visibility:

```json
"working_theories": [
  {
    "id": "theory-001",
    "title": "Constraint-Shift Pattern",
    "visibility": "public",     // ← this theory is public
    "thesis": "...",
    "confidence": 87
  },
  {
    "id": "theory-002",
    "title": "My Investment Thesis",
    "visibility": "private"     // ← this one never leaves the device
  }
]
```

---

## The On/Off Switch

The entire feed can be disabled instantly:

```json
"feed_controls": {
  "enabled": false    // ← feed returns 404, nothing exposed
}
```

When disabled:
- The endpoint returns HTTP 404
- No data is served
- No caching — callers must respect the 404
- Can be re-enabled at any time

Granular controls:

```json
"feed_controls": {
  "enabled": true,
  "sections_enabled": {
    "public_profile": true,
    "domain_expertise": true,
    "working_theories": false,   // ← temporarily hidden
    "contact_endpoint": true
  },
  "rate_limit": "10/hour",       // ← throttle crawlers
  "require_user_agent": true     // ← block anonymous scrapers
}
```

---

## AI-to-AI Handshake

When an AI agent wants to communicate with a PMS node on behalf of a user, it performs a handshake before sending the message:

### Step 1: Query public feed
```
GET https://tsnmedia.org/.well-known/pms.json
```

Returns public profile, communication preferences, preferred channel.

### Step 2: Request handshake (for contacts-level data)
```
POST https://tsnmedia.org/.well-known/pms-handshake
{
  "requester_node": "did:web:sender.com",
  "requester_pms": "https://sender.com/.well-known/pms.json",
  "purpose": "collaboration request",
  "message_preview": "first 100 chars of intended message"
}
```

### Step 3: Node responds
```json
{
  "status": "approved | pending | rejected",
  "shared_context": {
    // contacts-level data now shared with this requester
    "working_theories": [ ... ],
    "active_projects": [ ... ]
  },
  "preferred_response": {
    "channel": "telegram",
    "format": "direct, no preamble",
    "max_length": 500
  }
}
```

### Step 4: Message delivered with context
The sending AI now has the receiving user's preferences and context. It crafts the message accordingly. The receiving AI already knows who's asking and why.

Result: no context mismatch, no "who are you?" overhead, no mismatched tone.

---

## What This Enables

### For the receiver
- Every inbound AI-assisted message arrives pre-contextualised
- No explaining your preferences to every new contact
- AI can filter, prioritise, and draft responses using the sender's declared intent
- Spam is trivially filtered — anonymous senders without a PMS node get lowest priority

### For the sender
- Your AI knows how to address the recipient before composing the message
- Tone, format, channel, and timing preferences are machine-readable
- Relationship context is established before the first word is written

### For the network
- AI agents can route messages intelligently across the network
- Mutual PMS nodes establish trusted relationships automatically
- The network gets smarter as more nodes join — without any central coordinator

---

## Privacy Architecture

**The fundamental rule:** Private data never leaves the device.

```
Local device (always private):
├── committed_decisions
├── rejected_options  
├── belief_history
├── full working_theories (including private ones)
└── personal memory.json

Public endpoint (user-controlled):
├── public_profile (always opt-in)
├── selected working_theories (per-theory visibility)
├── domain_expertise summary
└── communication preferences

Contacts endpoint (verified contacts only):
├── active_projects (summary)
├── selected working_theories marked "contacts"
└── response preferences detail
```

No centralised server ever sees the private layer. The public endpoint is served from infrastructure the user controls (their own domain, a self-hosted server, or a privacy-preserving hosting service).

---

## Decentralised Identity

PMS-Connect uses **DIDs (Decentralised Identifiers)** as node identity:

```
did:web:tsnmedia.org          // domain-anchored, user controls
did:plc:abc123                // Bluesky/AT Protocol style
did:key:z6Mk...               // cryptographic key-based
```

No central registry. Identity is anchored to something the user controls. Verification is cryptographic.

This means:
- No company can revoke your PMS identity
- Node addresses are persistent even if you change hosting
- Cryptographic proof of who sent a message

---

## Integration With Existing Protocols

PMS-Connect doesn't replace existing protocols. It layers on top:

```
Email (SMTP)     ──→ UCIL ingestion ──→ PMS-Connect context
Telegram         ──→ UCIL ingestion ──→ PMS-Connect context  
Matrix/Element   ──→ native PMS-Connect (closest fit)
Nostr            ──→ native PMS-Connect (cryptographic identity already there)
AT Protocol      ──→ PMS-Connect extension (social graph maps to relationship graph)
```

Matrix is the best existing foundation — open, federated, end-to-end encrypted. A Matrix room with PMS-Connect metadata headers is likely the first implementation path.

---

## The Feed as Discovery Layer

The public JSON feed also serves as a **discovery mechanism**:

> "I'm looking for someone with expertise in X who is open to collaboration."

An AI agent can query PMS feeds across a network and find relevant people based on their declared domain expertise and open questions — without requiring a centralised directory.

```
Query: domain_expertise.expert contains "AI agents" AND open_questions contains "personalisation"
→ Returns: list of public PMS nodes matching criteria
→ AI composes contextualised outreach
→ Recipient's AI filters based on sender's PMS profile
```

This is professional networking done by AIs, grounded in structured identity, with full user control at every step.

---

## Roadmap

| Version | Milestone |
|---------|-----------|
| v0.1.0 | Public feed spec (`/.well-known/pms.json`) + visibility controls |
| v0.2.0 | AI handshake protocol + contacts-level sharing |
| v0.3.0 | DID integration + cryptographic verification |
| v0.4.0 | Discovery query protocol |
| v1.0.0 | Stable spec, reference implementation, Matrix integration |

---

## Prior Art Statement

This protocol was designed and published by Richard Lofthouse on April 29, 2026.

The specific combination of:
- User-controlled granular visibility per memory section
- Public JSON feed at `/.well-known/pms.json`
- On/off switch with instant revocation
- AI-to-AI handshake using PMS context before message delivery
- Decentralised identity anchoring (DID)
- Discovery via structured expertise queries

...as a unified protocol for AI-native human communication constitutes the original contribution of this work.

---

## Citation

```
Lofthouse, R. (2026). PMS-Connect: AI-Native Communication Protocol.
GitHub: https://github.com/techsocialnetwork/ideas
```

*© 2026 Richard Lofthouse / TSN Media — tsn@tsnmedia.org*
