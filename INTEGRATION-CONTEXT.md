# Integration Context: Agent Garage × TwoDigits Marketplace

> Stand: 2026-02-20
> Zweck: Schnell-Referenz für Claude-Sessions — kein erneutes Durchlesen aller Quellen nötig.

---

## Das große Bild

**Garage ist die Plattform, Marketplace liefert SDLC-Expertise.**
Der Marketplace wird in die Garage gemergt — nicht umgekehrt. Der Endnutzer chattet in Open WebUI und sieht nur Antworten. Alles andere ist unsichtbar.

---

## Was in der Agent Garage läuft (Stand heute)

| Komponente              | Status       | Beschreibung                                              |
| ----------------------- | ------------ | --------------------------------------------------------- |
| Open WebUI              | ✅ Läuft     | Chat-Interface, Personas, Session-Management              |
| n8n                     | ✅ Läuft     | Workflow-Orchestrierung, LangChain Agent Nodes            |
| Ollama                  | ✅ Läuft     | Lokale LLMs (llama3.2, qwen3-vl:8b)                       |
| Qdrant                  | ✅ Läuft     | Vektor-DB — **vorhanden, aber nicht genutzt**             |
| Jira (Docker)           | ✅ Läuft     | Self-hosted, via MCP-Atlassian verbunden                  |
| MCP-Atlassian           | ✅ Läuft     | Jira create/search/update/transition/comment              |
| Manager-Agent Pattern   | ✅ Läuft     | Supervisor → Sub-Agents (4 Agents)                        |
| User Story Creator      | ✅ Läuft     | Workflow 1: einfacher Story-Generator                     |
| Spec-Driven Developer   | ✅ Läuft     | Workflow 3: 11 Agents, generiert vollständige Specs (SDD) |
| Architecture Analyzer   | ✅ Läuft     | Workflow 4: Multimodal, analysiert Diagramme              |
| Logfile-Agent           | ✅ Läuft     | Workflow 2.4: Log-Analyse, Error Detection                |
| Bugreport-Agent         | ✅ Läuft     | Workflow 2.3: strukturierte Bug Reports                   |
| GitHub Actions (Claude) | ✅ Konfiguriert | Code Review + @claude Kommentar-Antworten — **vom Chat entkoppelt** |
| Memory Buffer           | ✅ Läuft     | Session-basiert, kein Cross-Session-Gedächtnis            |

**Verzeichnisstruktur:**
```
agent-garage/
├── docker-compose.yml      # Stack-Definition
├── n8n/backup/             # n8n Workflow-Exports (JSON)
├── openwebui/              # Open WebUI Konfiguration
├── shared/                 # Geteilte Daten zwischen Containern
├── logs/                   # Log-Dateien (test.log)
├── sdlc-gap-analysis.md    # Vollständige Gap-Analyse (Quelle der Wahrheit)
└── application-arch.png    # Architektur-Diagramm
```

---

## Was im Marketplace liegt (Stand heute)

**Pfad:** `/workspace/twodigits-marketplace`
**Branch:** `sdlc_expansion`

| Komponente                  | Status             | Beschreibung                                                |
| --------------------------- | ------------------ | ----------------------------------------------------------- |
| 5 SDLC-Phase-Orchestratoren | ⚠️ Stubs           | Requirements, Dev, Testing, Release, Ops — Markdown-Prompts |
| 21 Agenten-Definitionen     | ⚠️ Stubs           | Orchestratoren + Spezialisten pro Phase                     |
| 80+ Skill-Definitionen      | ✅ Vollständig     | SKILL.md mit Inhalt                                         |
| 14 MCP Server               | ⚠️ Stubs           | Python-Gerüste, ~40 Zeilen, nicht implementiert             |
| Claimification Plugin       | ✅ Produktionsreif | 9 Skills, 2 MCP Server, vollständige Python-Implementierung |
| human-in-the-core           | ✅ Registriert     | COBOL→Java Migration (15 Agents, 28 Commands)               |
| mainframe-discovery-toolkit | ✅ Registriert     | Automated Mainframe Analysis                                |
| task-manager                | ✅ Registriert     | Priority-based Task Management                              |

**SDLC-Framework im Marketplace:**
- **5 Phasen:** Requirements, Coding, Testing, Release, Operations
- **80+ Skills** als SKILL.md-Dateien (Single Source of Truth für CLI + Garage)
- **12 MCP Server Typen** definiert (als Stubs)
- Skills-Pfad: `twodigits-marketplace/sdlc/phases/*/SKILL.md` (ungefähr)

---

## Die 5 wichtigsten Bausteine für die Integration

### 1. `load_skill` Custom Code Tool (Aufwand: XS)
Ein einziges n8n Custom Code Tool (~20 Zeilen JS) das SKILL.md aus dem Marketplace-Repo liest.
- Jeder Phase-Agent bekommt dieses eine Tool
- Agent entscheidet kontextabhängig welchen Skill er braucht
- SKILL.md bleibt Single Source of Truth für CLI + Garage
- Neue Skills = neue Markdown-Datei, kein neuer n8n-Workflow

### 2. SDLC Supervisor (Aufwand: M)
Erweiterung des bestehenden Manager-Agent-Patterns auf 5 SDLC-Phasen.
- Bestehendes Muster: `Manager → [Jira, UserStory, Bugreport, Logfile]`
- Ziel: `SDLC Supervisor → [Requirements, Dev, Testing, Release, Ops]`
- Jeder Phase-Agent hat `load_skill` + phasenspezifische MCP-Tools

### 3. GitHub MCP Server (Aufwand: M)
GitHub REST API als MCP Server wrappen.
- PRs aus Chat erstellen
- PR-Status im Chat sehen
- GitHub Actions Claude Code Review mit Chat verbinden
- Branch-Erstellung aus Chat heraus

### 4. Elicitation Workflow (Aufwand: S)
Multi-Turn n8n Workflow für strukturierten Klärungsdialog.
- User gibt Feature-Idee ein
- Agent stellt Rückfragen bis vollständige Anforderung vorliegt
- Dann automatisch Jira-Push

### 5. Qdrant aktivieren (Aufwand: M)
Qdrant läuft, wird aber nicht genutzt. Drei Anwendungsfälle:
- Codebase indexieren → Code-Companion hat Codebase-Kontext
- Requirements indexieren → Cross-Phase Memory
- Runbook-RAG → Incident Response mit Wissen aus vergangenen Vorfällen

---

## Priorisierungsplan

### 🔴 P0 — Demo-ready
1. ✅ Git Submodule `marketplace/` in agent-garage eingebunden (sdlc_expansion branch)
2. ✅ `docker-compose.yml`: `./marketplace:/data/marketplace:ro` Volume-Mount hinzugefügt
3. ✅ `n8n/backup/workflows/5_Load_Skill.json` — Sub-Workflow liest SKILL.md aus `/data/marketplace`
4. ✅ `n8n/backup/workflows/5_SDLC_Supervisor.json` — Supervisor mit load_skill + 4 bestehenden Agents
   ⚠️  Nach Import: load_skill toolWorkflow-Referenz in n8n UI einmalig verbinden (REPLACE_AFTER_IMPORT)
5. ✅ Open WebUI Personas dokumentiert → `openwebui/PERSONAS-SETUP.md`
   ⚠️  Manuell in Open WebUI UI anlegen (6 Modelle: Supervisor + 5 Phasen)
6. ⬜ Elicitation Agent als n8n Multi-Turn-Dialog (S)
7. ⬜ GitHub MCP Server (M)
8. ⬜ Jira MCP vervollständigen — Epic-Kontext, Ticket lesen (S)

### 🟠 P1 — Vollständiges SDLC-Erlebnis
6. Qdrant Indexierungs-Pipeline (Codebase + Requirements) (M)
7. CI/CD MCP Server (GitHub Actions API) (M)
8. Confluence MCP (Stub → Implementation) (S)
9. Cross-Phase Memory über Qdrant (M)
10. Schlüssel-Skills implementieren: generate-user-stories, review-pr, generate-tests, root-cause-analysis

### 🟡 P2 — Enterprise
11. Prometheus/Grafana MCP
12. ArgoCD/Kubernetes MCP
13. LLM Router (Ollama ↔ Claude API)
14. SonarQube MCP, Feature Flag MCP

### 🟢 P3 — Vollständigkeit
15. Slack MCP, PagerDuty MCP
16. Phase 2 Plugin (Business Alignment), Phase 16/17 Plugins
17. Figma MCP

---

## Was NICHT gebaut werden muss

- Phase 5 Sprint Planning: Jira-Agent deckt Ticket-Zuweisung ab
- Phase 13 Go-Live Kommunikation: Marketing-Aufgabe, außerhalb Scope
- Phase 16/17 Compliance/Knowledge Sharing: P3
- Lokale Dev-Environments: Developer-seitig, kein Copilot-Feature
- Pair/Mob Programming: Geschieht in IDE

---

## Referenz-Dokumente

- Vollständige Gap-Analyse: `./sdlc-gap-analysis.md`
- SDLC Framework Overview: `./marketplace/sdlc/overview.md`
- Marketplace README: `./marketplace/README.md`
- Plugin Registry: `./marketplace/registry/`
- n8n Workflows: `./n8n/backup/workflows/`
- SKILL.md Dateien: `./marketplace/sdlc/{phase}/skills/{skill-name}/SKILL.md`
- Im n8n Container erreichbar unter: `/data/marketplace/sdlc/{phase}/skills/{skill-name}/SKILL.md`
