# System Requirement & Architecture Design Document

**Project Name:** Local Automated Document Redaction & Architecture Assessment System (LADRA)  
**Deployment Mode:** Private Local LAN / On-Premise  
**Document Version:** 0.1-DRAFT  
**Date:** 2026-07-23  

---

## 1. Executive Summary & Core Purpose

The **Local Automated Document Redaction & Architecture Assessment System (LADRA)** is an enterprise-grade, privacy-first software system designed to process sensitive architecture and API design documentation entirely within a private Local Area Network (LAN). 

LADRA guarantees data confidentiality by maintaining a strict operational boundary: raw confidential documents are never exposed to Large Language Models (LLMs) or external networks. Instead, a **Deterministic Local Processing Engine** desensitizes text and embedded diagram images before delivering sanitized artifacts to a **Local AI Agent Harness** (powered by Microsoft Agent Framework and locally hosted LLM engines such as vLLM, Ollama, or DeepSeek).

---

## 2. System Overview & Security Architecture

```text
                        [ LOCAL LAN / ON-PREMISE BOUNDARY ]
                                         │
┌────────────────────────────────────────┴────────────────────────────────────────┐
│                                                                                 │
│   ┌───────────────────────────┐                 ┌───────────────────────────┐   │
│   │   Folder A (Raw Inputs)   │                 │  Base Environment Context │   │
│   │   - Baseline v0.1.docx    │                 │  - Local Data Center Specs│   │
│   │   - Partner Modified v0.2 │                 │  - K8s Cluster Baseline   │   │
│   └─────────────┬─────────────┘                 │  - Network Topology Logs  │   │
│                 │                               └─────────────┬─────────────┘   │
│                 ▼                                             │                 │
│   ┌───────────────────────────────────────────┐               │                 │
│   │  1. Document Redaction & Change Engine    │               │                 │
│   │  - Track Changes & Fallback Text Diff     │               │                 │
│   │  - Text Mapping & RegEx Masking           │               │                 │
│   │  - Image OCR & Blackout Masking Pipeline  │               │                 │
│   └─────────────┬─────────────────────────────┘               │                 │
│                 │                                             │                 │
│                 ▼ Generates                                   │                 │
│   ┌───────────────────────────────────────────┐               │                 │
│   │   Folder B (Desensitized Artifacts)       │               │                 │
│   │   - Desensitized Docx v0.2.1              │               │                 │
│   │   - Change Log v0.2.1                     │               │                 │
│   └─────────────┬─────────────────────────────┘               │                 │
│                 │                                             │                 │
│                 └──────────────────────┬──────────────────────┘                 │
│                                        │ (Desensitized Data Only)               │
│                                        ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  2. Microsoft Agent Framework Harness Layer                             │   │
│   │  - Specialized Agents (Architecture, DevOps, Infra)                     │   │
│   │  - Custom Skill Support (Claude SKILL.md / MCP / Custom SOPs)           │   │
│   └────────────────────────────────────┬────────────────────────────────────┘   │
│                                        │ Local OpenAI-Compatible API            │
│                                        ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  3. Local LLM Server (vLLM / Ollama / DeepSeek Private Deploy)          │   │
│   └────────────────────────────────────┬────────────────────────────────────┘   │
│                                        │                                        │
│                                        ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────────┐   │
│   │  4. Web UI (React + Tailwind CSS)                                       │   │
│   │  - Mermaid.js Diagram Renderer  |  Human-in-the-Loop Clarification Chat   │   │
│   └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Core Functional Modules

### Module 1: Document Processing & Redaction Engine

* **Technology Stack:** Python (`FastAPI`, `python-docx`, `OpenCV`, `PaddleOCR` / `EasyOCR`) or C# (`.NET OpenXML`).

#### 1. Dual-Mode Document Comparison
* **Native Track Changes Parsing:** Parses OpenXML tags (`w:ins`, `w:del`, `w:comments`) to extract additions, deletions, and inline partner comments.
* **Fallback Text Diff Engine:** Executes character and sentence-level diffing algorithms (`diff_match_patch`) comparing baseline `v0.1` and modified `v0.2` to catch edits made without Word's "Track Changes" mode enabled.

#### 2. Text Desensitization
* **Lookup Reference Table:** Matches text against external lookup JSON tables (IP mappings, internal hostnames, confidential project names, API keys).
* **RegEx Masking Engine:** Applies regular expression filters for standard patterns (IPv4/IPv6, internal domain names, email addresses, tokens).

#### 3. Automated Image OCR & Blackout Pipeline
* **Extraction:** Extracts embedded diagrams, flowcharts, and screenshots from `v0.2.docx`.
* **OCR Text & Bounding Box Detection:** Executes local OCR (`PaddleOCR`/`EasyOCR`) to locate text strings and bounding box coordinates $(x_1, y_1, x_2, y_2)$.
* **Blackout Masking:** Matches detected text against the desensitization rules and draws solid black rectangles (`cv2.rectangle`) over sensitive regions.
* **Re-insertion:** Re-assembles and re-inserts redacted images into the final Word document output.

#### 4. Output Artifact Generation
* **`v0.2.1_desensitized.docx`**: Written to **Folder B**.
* **`ChangeLog_v0.2.1.docx` / `.md`**: Written to **Folder B**, detailing all modifications, original intent, and partner comments.

---

### Module 2: AI Agent Layer (Microsoft Agent Framework Harness)

* **Technology Stack:** Microsoft Agent Framework (Python/C# SDK), Local OpenAI-compatible API endpoint (`vLLM`, `Ollama`).

#### 1. Strict Data Ingestion Boundary
* Agents are hard-restricted to reading **only** desensitized artifacts from **Folder B** and baseline context files (`base_context`: K8s cluster specs, data center hardware baselines, network topology history).

#### 2. Specialized Multi-Agent Roles
* **Architecture & API Impact Agent:** Analyzes flowcharts, sequence diagrams, and API specifications. Identifies workflow bottlenecks, missing parameter definitions, and points of attention for third-party integration partners.
* **DevOps Agent:** Cross-references new API requirements with the baseline K8s environment. Computes suggested K8s configurations (`Deployment` YAML specs, CPU/Memory `requests`/`limits`, HPA scaling policies, PVC/PV storage needs).
* **Infra Agent:** Analyzes network boundaries. Generates a **Firewall Access Rule Matrix** (Source IP/Subnet, Target IP/Subnet, Port, Protocol, Sizing).

#### 3. Custom Skill Integration & "Self-Distillation"
* **Skill Compatibility:** Native support for loading custom **Claude SKILL.md** packages, Anthropic **Model Context Protocol (MCP)** tools, and custom Python `@tool` functions.
* **SOP & Experience Injection:** Allows domain experts to distill personal architecture rules, security checklists, and review templates into `SKILL.md` files that are dynamically loaded into the Agent Harness.

#### 4. Structured Diagram Formatting
* Enforces all network flows, deployment topologies, and API call sequences to be returned exclusively as valid **Mermaid.js** code blocks.

---

### Module 3: Web Frontend & Interactive UI

* **Technology Stack:** React, Tailwind CSS, Vite, `mermaid.js`, Lucide Icons.

#### 1. Dashboard & File Control
* Displays real-time processing status for Folder A to Folder B operations.
* Renders side-by-side comparison previews of raw vs. redacted images and change logs.

#### 2. Interactive Report & Diagram Viewer
* Categorized tabs:
  1. **Architecture & API Impact**
  2. **DevOps & K8s Spec**
  3. **Infra & Firewall Rules**
* Real-time vector rendering of Mermaid code into interactive diagrams (zoom, pan, export to SVG/PNG).

#### 3. Human-in-the-Loop (HITL) Clarification Chat
* Automatically flags `[NEED_CLARIFICATION]` tags whenever an Agent detects ambiguities or missing parameters in the documentation (e.g., *"API v0.2 introduces file upload; is persistent storage required?"*).
* Highlights these questions in an interactive chat UI.
* Reinjects user responses back into the Agent Harness to dynamically re-evaluate and update the assessment report.

---

## 4. Directory Structure Specification

```text
/my-local-system
├── /folder_a                     # [INPUT] Raw v0.1 & Partner modified v0.2 Word docs
├── /folder_b                     # [OUTPUT] Redacted v0.2.1 docx, ChangeLog & Redacted images
├── /base_context                 # [BASELINE] On-prem K8s, Network, Data Center specs
├── /desensitization_config       # [RULES] Mapping lookup tables & RegEx rules
│   ├── lookup_table.json
│   └── regex_patterns.json
├── /custom_skills                # [DISTILLED SKILLS] Claude SKILL.md & Custom Tools
│   ├── /architecture_review
│   │   ├── SKILL.md
│   │   └── checklist.md
│   └── /k8s_sizing_tool
│       └── tool.py
├── /backend                      # Python FastAPI Application
│   ├── /redaction_engine         # Track changes, diff, OCR & image masking
│   ├── /agent_harness            # Microsoft Agent Framework Harness & LLM Client
│   └── main.py
└── /frontend                     # React + Tailwind CSS Web Application
```

---

## 5. Key Non-Functional Requirements

| Category | Requirement Specification |
| --- | --- |
| **Network Isolation** | 100% executable offline within a local LAN. Zero external network/API calls permitted. |
| **LLM Endpoint** | OpenAI-compatible endpoint (`http://localhost:8000/v1` or local IP). |
| **Data Privacy** | Raw data in Folder A never leaves local disk; only redacted Folder B data is passed to the local LLM. |
| **Diagram Engine** | Frontend-rendered `mermaid.js` (no third-party chart generation services). |
| **Customizability** | Open skill loader architecture allowing direct addition of Python scripts, MCP servers, and Markdown SOPs. |

---

## 6. Next Steps & Continuation Plan (For Tomorrow)

1. **Phase 1: Backend Architecture Setup (`/backend`)**
   - Initialize FastAPI application structure.
   - Implement `/redaction_engine` core modules (`docx` track changes parser + `diff_match_patch` engine).
   - Setup OpenCV + OCR pipeline for image text detection and blackout masking.
2. **Phase 2: Desensitization Configuration & Folder Pipeline**
   - Create schema definitions for `lookup_table.json` and `regex_patterns.json`.
   - Build file system watchers/triggers from `/folder_a` to `/folder_b`.
3. **Phase 3: Microsoft Agent Framework Harness Integration (`/agent_harness`)**
   - Integrate Microsoft Agent Framework SDK with local OpenAI-compatible endpoint.
   - Implement the three specialized agent personas (Architecture, DevOps, Infra).
   - Configure skill loader for `SKILL.md` and custom tool functions.
4. **Phase 4: Frontend Web Application (`/frontend`)**
   - Setup React + Vite + Tailwind CSS project structure.
   - Develop Dashboard, Side-by-Side Image Redaction Viewer, Mermaid.js Renderer, and HITL Clarification Chat.
