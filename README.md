# IAAI Foundation Openware

**Open standards infrastructure for governed AI, RAG, and committee workflows.**

IAAI Foundation Openware is the **public, runnable mirror** of the IAAI Foundation stack.  
It provides the *machinery* for structured knowledge ingestion, retrieval-augmented reasoning, and standards governance — **without** bundling the Foundation’s private corpora, memberships, or quotas.

> **Run the machine locally.  
> Subscribe to the shared memory only if you need it.**

---

## What This Is

IAAI Foundation Openware is:

- A **complete local system** for:
  - RAG (retrieval-augmented generation)
  - provenance-first citation
  - structured governance workflows
  - SCXML-driven meetings, reviews, and ratification
- A **reference implementation** for how AI can support standards and coordination *without devolving into free-form chat*
- A **public fork** of a real production codebase, not a demo

You can:
- Run it entirely offline
- Use it with your own data
- Inspect and modify every moving part
- Integrate it into your own processes

---

## What This Is Not

This repository does **not** include:

- The IAAI Foundation’s canonical corpora
- Committee-only collections
- Moderation or curation pipelines
- Membership issuance, billing, or quotas
- Snapshot signing or ratification authority

Those remain part of the **IAAI Foundation services**.

This separation is intentional.

---

## Core Concepts

The system is built around three long-lived concepts that predate this fork and are preserved here:

### Memory Alpha
Developer- and work-artifact ingestion into RAG.  
Includes structured import of prior discussions, documents, and analysis.

### Memory Beta
Voice and autobiographical ingestion pipelines (e.g. call forwarding → text → RAG).  
Included for completeness and experimentation; private by default.

### Infinity State
The core UI and orchestration layer:
- Split-pane interface
- RAG-backed reasoning
- Workflow-driven interaction
- The foundation for both Q/A and governance modes

These are **inputs**, not marketing labels.

---

## Product Modes (Single Codebase)

The same codebase supports multiple modes via configuration:

- **openware_local**  
  Full local operation: RAG, workflows, UI, no external dependencies.

- **foundation_qa**  
  Public question/answer surface (requires Foundation access when enabled).

- **committee_manager**  
  Chat-less workflow manager for meetings, review, voting, and ratification.

Mode selection changes *capabilities*, not architecture.

---

## Governance by State Machine

A central design principle:

> **Standards are not written by chat. They are ratified by process.**

Governance flows are encoded as **state machines (SCXML)**:
- Explicit states
- Explicit transitions
- Explicit votes and decisions
- Auditable outputs

This avoids “AI consensus theater” and produces artifacts that can survive scrutiny.

---

## RAG Included, Batteries Optional

Openware ships with:
- Local RAG (vector + metadata)
- Provenance and citation tracking
- Hybrid retrieval (semantic + lexical)
- Deterministic workflows

Optional:
- Federation with IAAI Foundation retrieval endpoints
- Shared corpus snapshots
- Committee collaboration across organizations

Federation requires proof of domain control and a membership token.  
Local operation does not.

---

## Who This Is For

- Standards bodies and working groups
- Broadcast, media, and infrastructure engineers
- Organizations that need **governed AI**, not chatbots
- Teams who want transparency over “AI magic”
- Anyone evaluating how AI can assist coordination without replacing judgment

---

## Status

- Actively used in private and Foundation contexts
- Openware fork is intentionally conservative:
  - minimal abstractions
  - explicit contracts
  - no speculative features

This repository prioritizes **clarity over novelty**.

---

## Getting Started

> **Local first. Federation later.**

Basic steps:
1. Clone the repository
2. Run in local mode
3. Load your own documents
4. Experiment with workflows
5. Inspect the SCXML and provenance outputs

Detailed setup instructions live alongside the code.

---

## Relationship to IAAI Foundation

- This repository is **open**
- IAAI Foundation services are **governed and subscription-based**
- The two are designed to interoperate cleanly

Think of this repo as:

> *“The open reference engine behind a shared memory commons.”*

---

## License

See `LICENSE`.

---

## Further Reading

- **IAAI Foundation Charter:** `IAAI-FOUNDATION.md`  
  (Intent, boundaries, and conversion plan for this ecosystem)
