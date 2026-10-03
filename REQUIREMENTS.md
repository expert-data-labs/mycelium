# Mycelium — Product Requirements

> **A learning map, not a chat tree.**
> Each topic is a node with its own AI chat, and context flows down the branches as summaries.

| | |
|---|---|
| **Status** | Requirements finalised · tech stack not yet chosen |
| **Owner** | Pooja Singh |
| **Last updated** | 2026-10-03 |
| **Lineage** | Successor to the `graph-llm` prototype (React Flow canvas, FastAPI + Neo4j CRUD, Ollama agent) |

---

## 1. Purpose

### 1.1 Problem
Linear LLM chats force a bad trade-off: either earlier context is lost, or unrelated context leaks in.
Real research isn't a straight line. You branch, go deep, come back and connect ideas, and a single thread can't represent that.

### 1.2 Solution
Mycelium models learning as a **graph of topics**. Every node has its own chat.
A node's AI sees **only its own ancestor chain**, passed down as compact summaries, plus any references you add explicitly.
Over time the graph becomes a persistent, visual map of what you've learned.

### 1.3 Goals
1. **Personal tool.** Something the owner actually uses day to day for research and interview prep.
2. **Portfolio piece.** It shows graph modelling, context engineering and LLM integration, with a clean architecture worth explaining in interviews.
3. **Possible product later.** Don't make choices now that block multiple users, hosting or other LLM providers.

### 1.4 Target users
Researchers, students, professors, self-learners and interview candidates. In other words, anyone who goes down rabbit holes.
**v1 is single-user and local-first.**

### 1.5 Positioning against existing tools
Branching AI chat already exists (ChatGPT "Branch in new chat", Nodea, RabbitHoles AI, the tldraw branching template).
Mycelium differs in four ways:

| Others | Mycelium |
|---|---|
| Nodes are *replies* | Nodes are *topics*, so the graph reads as a knowledge map |
| Inherit the *raw transcript* | Inherit *summaries*, which stay cheap however deep the tree goes |
| Built for one session | Lasting graphs per subject, with links across graphs |
| Context is hidden | **Context preview**: you see and can edit what the AI receives |

---

## 2. Core concepts

| Term | Definition |
|---|---|
| **Graph** | A separate workspace, e.g. "System Design" or "Philosophy" |
| **Node** | A topic with a title, one chat thread and a summary |
| **Parent edge** | `CHILD_OF` relationship. **Exactly one parent per node** (a tree inside each graph). Defines inheritance |
| **Root node** | A node with no parent. A graph can have several roots |
| **Ancestor chain** | Path from the root to the node's parent |
| **Node summary** | Short, automatically maintained digest of a node's chat that the user can edit. It is what flows down to children |
| **Reference** *(v2)* | An explicit link that pulls another node's summary in as extra context. The target can be in the same graph or a different one |
| **Context bundle** | Everything sent to the LLM for one turn (see §4) |

---

## 3. Functional requirements

Priority: **M** = MVP (must have), **S** = v2 (should have), **C** = later (could have).

### 3.1 Graphs
| ID | Requirement | P |
|---|---|---|
| G1 | Create, rename and delete graphs; list all graphs | M |
| G2 | Open a graph to see its canvas | M |

### 3.2 Nodes & structure
| ID | Requirement | P |
|---|---|---|
| N1 | Create a root node, or a child of the selected node | M |
| N2 | Rename a node and edit its description | M |
| N3 | Each node has at most one parent (the backend enforces this) | M |
| N4 | Re-parent a node by dragging it onto another node. This changes its inherited context; cycles are rejected | M |
| N5 | Delete a node, choosing whether children are deleted with it or moved up to the deleted node's parent | M |
| N6 | Save canvas positions automatically | M |
| N7 | Search nodes by title, summary and message content | M |

### 3.3 Chat
| ID | Requirement | P |
|---|---|---|
| C1 | Clicking a node opens its chat panel | M |
| C2 | Chat history is saved per node | M |
| C3 | Responses stream in as they are generated | M |
| C4 | **Branch from a message:** choose any message and select "branch off" to create a child node seeded with it | M |
| C5 | Regenerate the last response | S |

### 3.4 Context inheritance (core)
| ID | Requirement | P |
|---|---|---|
| X1 | Every LLM call receives the context bundle described in §4. Sibling and cousin branches are **never** included | M |
| X2 | **Live inheritance:** changes to an ancestor's summary affect its descendants on their next turn | M |
| X3 | Node summaries update automatically after each chat turn | M |
| X4 | Users can edit a summary by hand. A manual edit stays until the user resets it | M |
| X5 | **Context preview** shows exactly what will be sent: each part, where it came from, and its token count | M |
| X6 | When context exceeds the token budget, it is trimmed predictably (see §4.2) and the preview shows what was trimmed | M |

### 3.5 References (v2)
| ID | Requirement | P |
|---|---|---|
| R1 | Add a reference from a node to any node in the same graph or another graph | S |
| R2 | A reference contributes **only the target's summary** (not the target's ancestors) | S |
| R3 | References are **not inherited** by children | S |
| R4 | References appear visually (dashed edge or pin), can be switched on or off, and are labelled in the context preview | S |
| R5 | If the referenced node is deleted, the reference is removed and the user is notified | S |

### 3.6 Later
| ID | Requirement | P |
|---|---|---|
| L1 | AI suggests subtopics to branch into next | S |
| L2 | Export a node or a branch as clean Markdown or PDF study notes | S |
| L3 | Visual cues on nodes: depth, how active the chat is, last visited | C |
| L4 | Add sources (PDFs, URLs) to a node | C |
| L5 | Multiple users, authentication, sharing | C |

---

## 4. Context bundle specification

### 4.1 Assembly order
```
1. System prompt              (app-level instructions)
2. Ancestor summaries         root → … → parent, each labelled with its title
3. References (v2)            summaries only, labelled "Reference: <graph>/<node>"
4. Current node summary       (if the chat is long enough to have one)
5. Recent messages            the latest turns of the current node, verbatim
6. User's new message
```

### 4.2 Token budget & trimming
- The budget is configurable for each model.
- Trimming order, from first trimmed to last: older verbatim messages → reference summaries → the most distant ancestor summaries.
- Never trimmed: the system prompt, the parent's summary, and the user's new message.

### 4.3 Example
```
Distributed Systems
├── Kafka
│   └── Consumer Groups
└── Consensus
    └── Raft            ← chatting here
```
Raft receives the summaries of Distributed Systems and Consensus, plus Raft's own chat.
**Nothing from the Kafka branch is included.**

---

## 5. Non-functional requirements

| Area | Requirement |
|---|---|
| **LLM-agnostic** | The LLM provider is a pluggable setting. Chat and summarisation can use different models |
| **Runs on modest hardware** | Development and use must work on an 8GB MacBook Air. Local models (~4B) are optional, not required |
| **Cost** | Runs entirely on free tiers or local models |
| **Privacy** | Data is stored locally by default. The UI shows which provider each call goes to |
| **Responsiveness** | First streamed token within ~2s on cloud providers. The canvas stays smooth with 200+ nodes |
| **Reliability** | Summary updates run asynchronously and never block or lose a chat message |
| **Ready for a product later** | A user/owner ID is in the data model from day one, even while there is a single user |
| **Portfolio quality** | Clean module boundaries, an architecture doc with diagrams, tests for the context builder, one-command local setup |

---

## 6. Out of scope (v1)
Multiple users, authentication, sharing or collaboration, mobile apps, deployment and hosting, payments, voice, image generation.

---

## 7. Success criteria (MVP)
1. The owner uses Mycelium for real system-design prep and actually prefers it to a plain chat for branching topics.
2. Ancestor isolation is proven by tests: a node never receives context from sibling branches.
3. A chat 5 levels deep keeps its context bundle within budget and stays coherent.
4. A 2-minute demo makes the idea obvious: create, branch, show the context preview, re-parent, and watch the context change.

---

## 8. Decisions log
| # | Decision | Rationale |
|---|---|---|
| D1 | Strictly one parent per node | Inheritance stays unambiguous; references cover cross-links |
| D2 | Live inheritance | The graph reflects current understanding, not stale snapshots |
| D3 | References are not inherited | Keeps context predictable and the budget under control |
| D4 | References allowed in the same graph and across graphs | Sibling branches are often the most relevant thing to reference |
| D5 | Summaries, not transcripts, are passed down | Scales with depth; works with small and free models |
| D6 | Single-user, local-first for v1 | Personal tool first; multi-user later without a rewrite |
| D7 | Start fresh, not a fork of `graph-llm` | Reuse lessons and ideas, not code |

---

## 9. Open for the tech-stack phase
- LLM: free cloud tier (Gemini Flash / Groq) for chat, optionally a small local model (Ollama, ~4B) for summaries
- Storage: Neo4j (as in the prototype) or alternatives
- Backend language and framework, how responses stream, and how summary jobs are queued
- Frontend: React Flow (from the prototype) and state management
