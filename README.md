# LADRA (Local Automated Document Redaction & Architecture Assessment System)

LADRA is an enterprise, privacy-first system designed to process sensitive architecture and API design documentation entirely within a private Local Area Network (LAN). 

It enforces a strict security boundary by using a **Deterministic Local Processing Engine** to desensitize text and embedded diagram images before delivering sanitized artifacts to a **Local AI Agent Harness** (powered by Microsoft Agent Framework and local LLM endpoints).

---

## 📚 Project Documentation

- **[System Requirement & Architecture Design Document](file:///Users/hank/Documents/projects/dev-design-ladara/docs/requirements/LADRA_System_Requirements.md)** — The complete functional, non-functional, and module specifications.
- **[Domain Vocabulary & Context (`CONTEXT.md`)](file:///Users/hank/Documents/projects/dev-design-ladara/CONTEXT.md)** — Ubiquitous language definition for concepts used in LADRA.

---

## 🏗 System Architecture Overview

```text
                        [ LOCAL LAN / ON-PREMISE BOUNDARY ]
                                         │
 ┌───────────────────────────┐           │         ┌───────────────────────────┐
 │   Folder A (Raw Inputs)   │           │         │  Base Environment Context │
 └─────────────┬─────────────┘           │         └─────────────┬─────────────┘
               ▼                         │                       │
 ┌───────────────────────────┐           │                       │
 │ 1. Redaction & Diff Engine│           │                       │
 └─────────────┬─────────────┘           │                       │
               ▼ Generates               │                       │
 ┌───────────────────────────┐           │                       │
 │Folder B (Desensitized)    │───────────┼───────────────────────┘
 └───────────────────────────┘           │
                                         ▼
                         ┌───────────────────────────────┐
                         │ 2. Microsoft Agent Harness    │
                         └───────────────┬───────────────┘
                                         ▼
                         ┌───────────────────────────────┐
                         │ 3. Local LLM Server (vLLM)    │
                         └───────────────┬───────────────┘
                                         ▼
                         ┌───────────────────────────────┐
                         │ 4. Web UI (React + Tailwind)  │
                         └───────────────────────────────┘
```

---

## 📁 System Directory Structure

```text
/my-local-system
├── /folder_a                     # [INPUT] Raw v0.1 & Partner modified v0.2 Word docs
├── /folder_b                     # [OUTPUT] Redacted v0.2.1 docx, ChangeLog & Redacted images
├── /base_context                 # [BASELINE] On-prem K8s, Network, Data Center specs
├── /desensitization_config       # [RULES] Mapping lookup tables & RegEx rules
├── /custom_skills                # [DISTILLED SKILLS] Claude SKILL.md & Custom Tools
├── /backend                      # Python FastAPI Application
└── /frontend                     # React + Tailwind CSS Web Application
```

---

## 🚀 Tomorrow's Implementation Roadmap

1. **Phase 1:** Backend FastAPI setup & Document Redaction Engine (`w:ins`/`w:del` Track Changes parser + `diff_match_patch` fallback).
2. **Phase 2:** Image OCR & Blackout Masking Pipeline (`PaddleOCR`/`EasyOCR` + `OpenCV`).
3. **Phase 3:** Microsoft Agent Framework Harness & 3 Specialized Agent Personas.
4. **Phase 4:** React + Tailwind CSS Frontend UI with Mermaid.js Vector Renderer & HITL Clarification Chat.
