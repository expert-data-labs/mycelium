# Mycelium 🍄

> Non-linear LLM conversations where context flows along the branches of a knowledge graph.

Linear chat threads lose context, or drag in everything. Mycelium treats every conversation as a **node** in a graph.
A new node inherits context **only from its ancestors** (the path you chose), not from unrelated branches.
Over time the graph becomes a visual map of your learning journey.

**For:** researchers, students, professors, and anyone who goes down rabbit holes.

## Core idea: hierarchical context

```
        [Distributed Systems]
          /             \
     [Kafka]          [Consensus]
      /     \               \
[Partitions] [Consumer Groups] [Raft]
```

Chatting in **Raft** sends the LLM: summaries of *Distributed Systems* → *Consensus* → *Raft*'s own history.
It does **not** send anything about *Kafka*.

## Planned architecture (indicative — stack not final)

```
React + React Flow  (graph canvas + chat panel)
        ↓ REST / streaming
FastAPI  (graph API, context builder, LLM agent)
        ↓                 ↓
Neo4j (nodes, edges,   Ollama / LLM provider
       messages, summaries)
```

## Status

Requirements finalised — see [REQUIREMENTS.md](REQUIREMENTS.md). Tech stack is next.
Evolves from the `graph-llm` prototype (React Flow UI, FastAPI + Neo4j CRUD, Ollama agent).
