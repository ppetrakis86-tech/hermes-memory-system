# Hermes Memory System (HMS)

**A structured, multi-layer memory architecture for Large Language Model (LLM) conversations.**

> Project inception: February 2026  
> Author: Petros Petrakis  
> License: MIT

---

## The Problem

LLMs have no persistent memory between sessions. Current workarounds — conversation context, native "memory" features, retrieval-augmented generation (RAG) — each solve part of the problem but introduce their own limitations:

- Context windows are finite and expensive in tokens
- Native memory features (where they exist) have character/entry limits and limited structure
- RAG requires external infrastructure
- None of them offer the user **direct, structured control** over what is remembered, how it is stored, and when it is retrieved

Hermes Memory System was designed to solve this through a **layered, token-efficient, user-controlled architecture** built entirely within the LLM's existing interface — no external databases, no APIs, no infrastructure.

---

## Core Concept

HMS treats memory as a **three-layer system**:

### Layer 1 — Automatic Memory
The LLM's native automatic memory feature (where available). Left to operate normally, but **guided** by Layer 2 pointers so that summaries are more targeted and useful.

### Layer 2 — Manual Bridge
A small set of structured pointer entries written to the LLM's manual memory. These are not full memory entries — they are **navigation anchors** that tell the system where to find detailed information and when to retrieve it.

Bridge entries follow a strict template to maximize information density within limited manual memory slots:
- Structural pointers to vault entries
- ID references for targeted retrieval
- Usage instructions for the LLM

The Bridge is managed exclusively by the Refinery component (see below) — never edited manually mid-conversation.

### Layer 3 — Hermes Vault
One or more **dedicated conversation threads** used purely as structured storage. Memory blocks are written here with:
- Unique IDs (two-letter category prefix + sequential number, e.g. `SA01`, `GN03`)
- Tags for targeted search
- Categories for project separation
- Both a **plain facts list** and a **narrative version** of each entry (optimized for snippet retrieval)

Vault entries are retrieved on demand via conversation search — the LLM reads snippets from its search tool, not the full vault, keeping token usage low.

---

## Key Design Principles

**Lazy loading** — vault content is not loaded into every session. It is retrieved only when the conversation requires it, via explicit retrieval commands.

**Snippet optimization** — entries are structured so that search tool snippets (short previews returned by search) contain enough information to be useful without loading the full entry.

**Token efficiency** — vault operations use a lightweight model configuration. Full reasoning is reserved for complex tasks.

**User control** — the user decides what is stored, how it is categorized, and when it is retrieved. The system does not autonomously modify memory.

**Patch system** — memory entries can be updated with targeted text-based patches (add / remove / replace specific content by ID) without rewriting entire entries. This keeps memory accurate over time without token-expensive full rewrites.

---

## Multi-Model Implementation

HMS has been implemented and tested across three model types with different native memory capabilities:

**Model Type A** — No native memory. Full manual implementation. All memory operations are explicit and user-managed. Highest control, highest manual effort.

**Model Type B** — Native dynamic memory with direct write capability. The model can write and refresh its own memory entries automatically. HMS adds structure and IDs on top of this native capability, reducing the need for manual intervention.

**Model Type C** — Native automatic memory (summary-based) combined with manual memory slots and conversation search. Full HMS implementation: automatic memory guided by Bridge pointers, vault storage via dedicated threads, structured retrieval via search snippets, and patch-based updates.

The same architectural principles apply across all three. The implementation adapts to what each model natively supports.

---

## The Refinery

The Refinery is a separate component responsible for:
- Formatting raw information into structured vault entries
- Assigning and tracking IDs across all categories
- Maintaining ID STATE (a registry of the latest ID in each category)
- Comparing new information against existing entries to avoid duplication
- Generating Bridge pointer entries in the correct template format
- Processing memory patches

The Refinery runs on a higher-capability model configuration. Vault storage itself uses a lighter configuration to minimize token cost.

---

## Project Separation

HMS supports multiple parallel projects within the same system. Each project has:
- Its own two-letter ID prefix
- Its own vault section
- Its own Bridge pointer(s)
- Independent ID STATE tracking

This allows the system to scale without categories bleeding into each other, and allows selective loading — only the relevant project's memory is retrieved for any given conversation.

---

## Why This Matters

This architecture demonstrates that **structured, persistent, token-efficient memory for LLMs** can be built without external infrastructure, using only:
- The LLM's native conversation interface
- Its memory features (automatic and/or manual, depending on the model)
- Its search/retrieval tools
- A consistent naming and formatting convention

The result is a system where the user maintains full ownership and control of their AI memory — portable, inspectable, and independent of any single model or platform.

---

## Status

Active. Under continuous development.  
First implemented: February 2026.

---

*Hermes Memory System — designed and developed by Petros Petrakis, 2026.*
