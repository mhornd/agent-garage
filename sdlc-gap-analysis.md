# Gap-Analyse: Agent Garage × SDLC Marketplace

## Was fehlt für ein vollständiges End-to-End SDLC-Erlebnis?

> Perspektive: Endbenutzer chattet in Open WebUI. Alles andere ist unsichtbar.
> Methode: SDLC Marketplace wird **in** die Agent Garage gemergt — Garage ist die Plattform, Marketplace liefert SDLC-Expertise.
> Stand: 2026-02-19

---

## Ausgangslage

### Was die Agent Garage mitbringt (Plattform)

| Komponente              | Status          | Beschreibung                                        |
| ----------------------- | --------------- | --------------------------------------------------- |
| Open WebUI              | ✅ Läuft        | Chat-Interface, Personas, Session-Management        |
| n8n                     | ✅ Läuft        | Workflow-Orchestrierung, LangChain Agent Nodes      |
| Ollama                  | ✅ Läuft        | Lokale LLMs (llama3.2, qwen3-vl:8b)                 |
| Qdrant                  | ✅ Läuft        | Vektor-DB — vorhanden, aber **nicht genutzt**       |
| Jira (Docker)           | ✅ Läuft        | Self-hosted, via MCP-Atlassian verbunden            |
| MCP-Atlassian           | ✅ Läuft        | Jira create/search/update/transition/comment        |
| Manager-Agent Pattern   | ✅ Läuft        | Supervisor → Sub-Agents (4 Agents)                  |
| User Story Creator      | ✅ Läuft        | Workflow 1: einfacher Story-Generator               |
| Spec-Driven Developer   | ✅ Läuft        | Workflow 3: 11 Agents, generiert vollständige Specs |
| Architecture Analyzer   | ✅ Läuft        | Workflow 4: Multimodal, analysiert Diagramme        |
| Logfile-Agent           | ✅ Läuft        | Workflow 2.4: Log-Analyse, Error Detection          |
| Bugreport-Agent         | ✅ Läuft        | Workflow 2.3: strukturierte Bug Reports             |
| GitHub Actions (Claude) | ✅ Konfiguriert | Code Review + @claude Kommentar-Antworten           |
| Memory Buffer           | ✅ Läuft        | Session-basiert, kein Cross-Session-Gedächtnis      |

### Was der Marketplace mitbringt (Expertise)

| Komponente                  | Status             | Beschreibung                                                |
| --------------------------- | ------------------ | ----------------------------------------------------------- |
| 5 SDLC-Phase-Orchestratoren | ⚠️ Stubs           | Requirements, Dev, Testing, Release, Ops — Markdown-Prompts |
| 21 Agenten-Definitionen     | ⚠️ Stubs           | Orchestratoren + Spezialisten pro Phase                     |
| 80+ Skill-Definitionen      | ✅ Vollständig     | SKILL.md mit Inhalt                                         |
| 14 MCP Server               | ⚠️ Stubs           | Python-Gerüste, ~40 Zeilen, nicht implementiert             |
| Claimification Plugin       | ✅ Produktionsreif | 9 Skills, 2 MCP Server, vollständige Python-Implementierung |

---

## Die Benutzer-Journey: SDLC in Open WebUI

Der Nutzer öffnet Open WebUI. Er tippt. Er sieht Antworten. Was passiert dahinter ist irrelevant für ihn. Die Frage ist: **Wo funktioniert das heute, wo nicht?**

---

## Gap-Analyse: Phase für Phase

### PHASE 1 — Ideation & Discovery

> _"Wir haben das Problem X. Ich brauche eine neue Funktion."_

| Schritt                                   | Heute | Garage allein                | Nach Merge               | Was fehlt                            |
| ----------------------------------------- | ----- | ---------------------------- | ------------------------ | ------------------------------------ |
| Feature-Idee beschreiben                  | ✅    | User Story Creator           | Requirements-Agent       | —                                    |
| Strukturierter BA-Dialog (Klärungsfragen) | ❌    | Kein Workflow                | Elicitation-Agent (Stub) | **n8n-Workflow für Dialogue-Loop**   |
| Feature-Idee → strukturiertes Dokument    | ⚠️    | SDD macht das für Tech-Specs | Transformer-Agent (Stub) | **Skill-Implementierung + n8n-Node** |
| Stakeholder-Sentiment analysieren         | ❌    | Nichts                       | Stub vorhanden           | **Skill-Implementierung**            |
| Ideation-Sitzung moderieren               | ❌    | Nichts                       | Nichts                   | **Neues Plugin nötig**               |

**Fazit Phase 1:** Manager-Agent + User Story Creator decken die Grundlage ab. Fehlt: strukturierter Klärungsdialog (mehrere Gesprächsrunden zu einer Anforderung). Das ist der kritischste Demo-Step.

---

### PHASE 2 — Business Alignment & Prioritization

> _"Macht das Feature businessmäßig Sinn? Wann kommt es?"_

| Schritt                            | Heute | Nach Merge                                  | Was fehlt                                    |
| ---------------------------------- | ----- | ------------------------------------------- | -------------------------------------------- |
| Business Case erstellen            | ❌    | Nichts im Marketplace                       | **Neues Plugin + Skill**                     |
| ROI-Modellierung                   | ❌    | Nichts                                      | **Neues Plugin + Skill**                     |
| Feature priorisieren (RICE/MoSCoW) | ❌    | `prioritize-features` Stub in Release-Phase | **Skill-Implementierung + Jira-Integration** |
| Roadmap platzieren                 | ❌    | `generate-roadmap` Stub                     | **Skill-Implementierung**                    |
| Kapazitätsplanung                  | ❌    | Nichts                                      | **Neues Plugin + Skill**                     |

**Fazit Phase 2:** Vollständig ungedeckt. Für die Demo nicht kritisch — Business-Entscheidungen passieren außerhalb des SDLC-Copilots. Mittelfristig relevant für komplettes Erlebnis.

---

### PHASE 3 — Requirements Engineering

> _"Was genau soll gebaut werden? User Stories, Akzeptanzkriterien, Jira."_

| Schritt                                  | Heute | Nach Merge                           | Was fehlt                                           |
| ---------------------------------------- | ----- | ------------------------------------ | --------------------------------------------------- |
| User Stories generieren                  | ✅    | User-Story-Agent + Marketplace-Skill | MCP-Connection in Marketplace-Skill-Nodes           |
| Akzeptanzkriterien (Given/When/Then)     | ✅    | User-Story-Agent macht das bereits   | Qualität durch Validator-Agent verbessern           |
| → Jira pushen                            | ✅    | Jira-Agent (MCP Atlassian)           | —                                                   |
| Anforderungen aus Dokumenten extrahieren | ❌    | `extract-requirements` Stub          | **Skill-Implementierung + Qdrant für Dokument-RAG** |
| Konflikte erkennen                       | ❌    | `detect-conflicts` Stub              | **Skill-Implementierung**                           |
| Gap-Analyse                              | ❌    | `gap-analysis` Stub                  | **Skill-Implementierung**                           |
| Backlog Refinement                       | ❌    | Nichts                               | **n8n-Workflow + Jira-Integration**                 |
| Confluence-Seiten lesen/schreiben        | ❌    | Stub vorhanden                       | **MCP Confluence implementieren**                   |

**Fazit Phase 3:** Kernfunktion (Stories + Jira) läuft. Tiefere Anforderungsanalyse, Dokumentenextraktion, Confluence-Integration fehlt.

---

### PHASE 4 — Architecture & Design

> _"Wie bauen wir das? Welche Komponenten, welche APIs?"_

| Schritt                           | Heute | Nach Merge                                         | Was fehlt                             |
| --------------------------------- | ----- | -------------------------------------------------- | ------------------------------------- |
| Architekturdiagramm analysieren   | ✅    | Architecture Analyzer (Workflow 4, multimodal)     | —                                     |
| Implementierungsplan aus Diagramm | ✅    | Architecture Analyzer Stage 2                      | —                                     |
| Technisches Design-Dokument       | ✅    | SDD generiert OpenAPI, Datenfluss, Projektstruktur | —                                     |
| ADR erstellen                     | ❌    | Nichts                                             | **Neuer Skill + n8n-Node**            |
| Threat Modeling                   | ❌    | Nichts                                             | **Neuer Skill + n8n-Node**            |
| DB-Schema-Design                  | ⚠️    | SDD macht Datenfluss, kein explizites Schema       | **Verbesserung SDD oder neuer Skill** |
| UX/UI Design (Figma)              | ❌    | Nichts                                             | **Figma MCP Server**                  |
| Design Review                     | ❌    | Nichts                                             | **Neuer Workflow**                    |

**Fazit Phase 4:** Architecture Analyzer + SDD decken den Kern ab. ADR-Generator und Threat-Modeling sind die wichtigsten Ergänzungen.

---

### PHASE 5 — Sprint Planning & Preparation

> _"Wer macht was in diesem Sprint?"_

| Schritt                      | Heute | Nach Merge                       | Was fehlt                              |
| ---------------------------- | ----- | -------------------------------- | -------------------------------------- |
| Sprint Planning unterstützen | ❌    | Jira-Agent kann Tickets zuweisen | **n8n-Workflow für Sprint-Kontext**    |
| Branch-Strategie empfehlen   | ❌    | Nichts                           | **Neuer Skill + Source-Control-MCP**   |
| Dev-Environment Setup        | ❌    | Nichts                           | **Außerhalb Scope (Developer-seitig)** |

**Fazit Phase 5:** Großteils manueller Prozess. Jira-Agent kann Tickets zuweisen, der Rest ist außerhalb des Copilot-Scopes.

---

### PHASE 6 — Implementation / Coding

> _"Ich schreibe Code. Hilf mir."_

| Schritt                           | Heute | Nach Merge                                  | Was fehlt                                       |
| --------------------------------- | ----- | ------------------------------------------- | ----------------------------------------------- |
| Jira-Ticket lesen (Kontext holen) | ✅    | Jira-Agent (search/get)                     | —                                               |
| Code aus Anforderungen generieren | ⚠️    | SDD → Spec, aber kein Code-Generator in n8n | **Code-Companion Skill-Implementierung**        |
| Bug fixen                         | ❌    | Logfile-Agent erkennt Bugs, kein Fix        | **`fix-bug` Skill + Code-Kontext (Qdrant RAG)** |
| Code erklären                     | ❌    | `explain-codebase` Stub                     | **Skill + Codebase-Indexierung in Qdrant**      |
| Dokumentation generieren          | ❌    | `generate-docs` Stub                        | **Skill-Implementierung**                       |
| Pair-Programming Support          | ❌    | `pair-program` Stub                         | **Interaktiver Workflow nötig**                 |
| Branch erstellen                  | ❌    | Nichts                                      | **Source-Control MCP implementieren**           |
| CI/CD-Pipeline generieren         | ❌    | `generate-pipeline` Stub                    | **Skill + Template-System**                     |

**Fazit Phase 6:** Die "echte" Implementierung passiert in der IDE (Claude Code). Der Copilot kann Kontext liefern (Jira) und Specs generieren (SDD). Code-im-Chat ist sekundär — der Wert liegt in der Orchestrierung.

---

### PHASE 7 — Code Review & Quality Assurance

> _"Ist mein Code gut? PR erstellen, reviewen lassen."_

| Schritt                               | Heute | Nach Merge                              | Was fehlt                          |
| ------------------------------------- | ----- | --------------------------------------- | ---------------------------------- |
| PR automatisch reviewen               | ✅    | GitHub Actions (claude-code-review.yml) | Nicht in Open WebUI sichtbar       |
| PR aus Chat erstellen                 | ❌    | Nichts                                  | **GitHub MCP Server**              |
| PR-Status in Chat anzeigen            | ❌    | Nichts                                  | **GitHub MCP Server**              |
| Statische Analyse-Ergebnisse anzeigen | ❌    | `code-quality` Stub                     | **SonarQube MCP Server**           |
| Dependency Scan                       | ❌    | Nichts                                  | **Security-MCP (Snyk/Dependabot)** |
| Review-Kommentare via Chat            | ⚠️    | GitHub Actions reagiert auf @claude     | Kein aktiver Pull aus n8n heraus   |

**Fazit Phase 7:** GitHub Actions mit Claude Code Review läuft, ist aber vom Chat-Interface entkoppelt. Das Schlüsselstück: **GitHub MCP Server**, der aus n8n heraus PRs erstellen und Status abfragen kann.

---

### PHASE 8 — CI/CD Pipeline

> _"Läuft der Build? Sind alle Tests grün?"_

| Schritt                   | Heute | Nach Merge                                | Was fehlt                                          |
| ------------------------- | ----- | ----------------------------------------- | -------------------------------------------------- |
| Pipeline triggern         | ❌    | Stub vorhanden                            | **CI/CD MCP Server (GitHub Actions API)**          |
| Pipeline-Status abfragen  | ❌    | Stub vorhanden                            | **CI/CD MCP Server**                               |
| Build-Fehler analysieren  | ⚠️    | Logfile-Agent kann Build-Logs analysieren | **CI/CD MCP → Log-Weiterleitung an Logfile-Agent** |
| Quality Gates (SonarQube) | ❌    | Stub vorhanden                            | **SonarQube MCP Server**                           |
| Container bauen/scannen   | ❌    | Nichts                                    | **Außerhalb Scope**                                |
| IaC validieren            | ❌    | Nichts                                    | **Terraform/Checkov MCP (später)**                 |

**Fazit Phase 8:** Pipeline-Status im Chat wäre hoher Wert. Kern: CI/CD MCP implementieren, der GitHub Actions API wrапpt.

---

### PHASE 9 — Dedizierte Testphasen

> _"Tests schreiben, ausführen, Coverage messen."_

| Schritt                  | Heute | Nach Merge                | Was fehlt                          |
| ------------------------ | ----- | ------------------------- | ---------------------------------- |
| Test-Plan generieren     | ❌    | `generate-test-plan` Stub | **Skill-Implementierung**          |
| Unit Tests generieren    | ❌    | `generate-tests` Stub     | **Skill-Implementierung**          |
| E2E Tests generieren     | ❌    | `generate-e2e-tests` Stub | **Skill + Playwright-Integration** |
| Tests ausführen          | ❌    | `run-tests` Stub          | **CI/CD MCP**                      |
| Coverage-Report anzeigen | ❌    | `coverage-analysis` Stub  | **Codecov/Jacoco MCP**             |
| Performance Tests        | ❌    | `performance-test` Stub   | **k6 MCP**                         |
| Security Tests (DAST)    | ❌    | `security-test` Stub      | **OWASP ZAP / Nuclei**             |
| Flaky Tests fixen        | ❌    | `fix-flaky-tests` Stub    | **Skill-Implementierung**          |

**Fazit Phase 9:** Test-Generierung (Prompts/Skills) ist umsetzbar sobald SDD-Output als Input genutzt wird. Test-Ausführung braucht CI/CD-MCP.

---

### PHASE 10 — Release Management

> _"Feature fertig. Release vorbereiten, deployen."_

| Schritt                    | Heute | Nach Merge                   | Was fehlt                               |
| -------------------------- | ----- | ---------------------------- | --------------------------------------- |
| Changelog generieren       | ❌    | `changelog` Stub             | **Skill + GitHub MCP (Commit-History)** |
| Release Notes erstellen    | ❌    | `release-notes` Stub         | **Skill-Implementierung**               |
| Rollback-Plan              | ❌    | `rollback-plan` Stub         | **Skill-Implementierung**               |
| Feature Flag konfigurieren | ❌    | `feature-flag-strategy` Stub | **LaunchDarkly/Unleash MCP**            |
| Deployment starten         | ❌    | Stub vorhanden               | **ArgoCD/Kubernetes MCP**               |
| Go/No-Go Checklist         | ❌    | Nichts                       | **Neuer Workflow**                      |

**Fazit Phase 10:** Changelog und Release Notes sind reine LLM-Aufgaben (kein MCP nötig wenn GitHub-History als Context). Deployment braucht ArgoCD MCP.

---

### PHASE 11 — Deployment & Operations

> _"Das Feature ist live. Läuft alles?"_

| Schritt                     | Heute | Nach Merge     | Was fehlt                  |
| --------------------------- | ----- | -------------- | -------------------------- |
| Kubernetes-Status abfragen  | ❌    | Stub vorhanden | **Kubernetes MCP**         |
| Deployment starten/rollback | ❌    | Stub vorhanden | **ArgoCD MCP**             |
| Secrets managen             | ❌    | Nichts         | **Vault MCP (später)**     |
| Production Smoke Test       | ❌    | Nichts         | **CI/CD MCP + Playwright** |

**Fazit Phase 11:** Braucht ArgoCD/Kubernetes MCP — dasselbe wie Phase 10.

---

### PHASE 12 — Observability & Monitoring

> _"Gibt es Probleme in Production?"_

| Schritt             | Heute | Nach Merge                                  | Was fehlt                                  |
| ------------------- | ----- | ------------------------------------------- | ------------------------------------------ |
| Log-Analyse         | ✅    | Logfile-Agent (liest `/data/logs/test.log`) | Auf echte Log-Quellen erweitern (Loki/ELK) |
| Metriken abfragen   | ❌    | Stub vorhanden                              | **Prometheus/Grafana MCP**                 |
| Alerts anzeigen     | ❌    | Stub vorhanden                              | **Alertmanager MCP**                       |
| Distributed Tracing | ❌    | Nichts                                      | **Jaeger/Tempo MCP**                       |
| SLO-Status          | ❌    | `slo-management` Stub                       | **Skill + Grafana MCP**                    |
| Anomalie-Erkennung  | ❌    | `anomaly-detection` Stub                    | **Skill + Metriken-MCP**                   |

**Fazit Phase 12:** Logfile-Agent ist der Anfang. Erweiterung auf echte Monitoring-Systeme via MCP ist der nächste Schritt. Sehr hoher Wert für Operations-Teams.

---

### PHASE 13 — Go-Live & Customer Engagement

> _"Feature ausrollen, Nutzer informieren."_

| Schritt                   | Heute | Nach Merge | Was fehlt                            |
| ------------------------- | ----- | ---------- | ------------------------------------ |
| Internal Announcement     | ❌    | Nichts     | **Kommunikations-Skill (Slack MCP)** |
| Progressive Rollout       | ❌    | Nichts     | **Feature Flag MCP**                 |
| Support-Artikel erstellen | ❌    | Nichts     | **Confluence MCP + Skill**           |

**Fazit Phase 13:** Für Demo-Zwecke nicht kritisch. Slack MCP wäre ein großes UX-Feature.

---

### PHASE 15 — Incident Management & SRE

> _"Production ist down. Was jetzt?"_

| Schritt                            | Heute | Nach Merge                        | Was fehlt                                |
| ---------------------------------- | ----- | --------------------------------- | ---------------------------------------- |
| Incident erkennen & klassifizieren | ⚠️    | Logfile-Agent + Bugreport-Agent   | `classify-incident` Skill implementieren |
| Root Cause Analysis                | ⚠️    | Logfile-Agent macht Error-Analyse | `root-cause-analysis` Skill vertiefen    |
| Incident-Kanal öffnen (Slack)      | ❌    | Nichts                            | **Slack MCP**                            |
| Rollback triggern                  | ❌    | Nichts                            | **ArgoCD MCP**                           |
| Post-Mortem generieren             | ❌    | `incident-summary` Stub           | **Skill-Implementierung**                |
| PagerDuty/OpsGenie                 | ❌    | Stub vorhanden                    | **Incident-MCP implementieren**          |

**Fazit Phase 15:** Logfile-Agent + Bugreport-Agent sind gute Grundlage. Rollback braucht ArgoCD MCP. Post-Mortem ist reine LLM-Aufgabe.

---

## Übergreifende Lücken (Cross-Cutting)

Diese fehlen unabhängig von einzelnen Phasen:

### 1. SDLC Supervisor — DAS kritischste fehlende Element

Das Manager-Agent-Pattern existiert für 4 Sub-Agents. Es fehlt ein **SDLC Supervisor** der alle 5 Phasen kennt und koordiniert:

```
Was heute existiert:
Manager → [Jira, UserStory, Bugreport, Logfile]

Was gebraucht wird:
SDLC Supervisor → [Requirements, Development, Testing, Release, Ops]
                    ↑ jeweils mit eigenen Sub-Agents und MCP-Tools
```

**Implementierungsaufwand:** Mittel — das Muster existiert, es muss auf 5 Phasen erweitert werden.

### 2. Marketplace-Skills via `load_skill`-Tool

80+ Skill-Definitionen existieren als SKILL.md-Dateien im selben Repository. Da alles in einem Repo liegt, können Phase-Agents diese zur Laufzeit laden — statt alle Skills in den System-Prompt einzubetten oder pro Skill einen eigenen Sub-Workflow zu bauen.

**Funktionsprinzip:** Jeder Phase-Agent bekommt ein einziges zusätzliches Tool: `load_skill`. Der Agent entscheidet kontextabhängig welchen Skill er braucht, ruft `load_skill("generate-user-stories")` auf, bekommt den Skill-Prompt zurück und wendet ihn an. Progressive Disclosure — der Kontext bleibt klein.

```
Requirements-Agent
  ├── Tool: load_skill(skill_name)   ← liest SKILL.md aus Repo-Dateisystem
  │         → gibt Prompt + Input/Output-Schema zurück
  ├── Tool: jira_mcp
  └── Tool: confluence_mcp

Dev-Agent
  ├── Tool: load_skill(skill_name)   ← selber Loader, andere Skills
  ├── Tool: github_mcp
  └── Tool: ci_cd_mcp
```

**Implementierung in n8n:** `load_skill` ist ein **Custom Code Tool** (~20 Zeilen JS) der die entsprechende SKILL.md aus dem Dateisystem liest. Einmaliger Baustein — nicht 80x wiederholt.

**Vorteil gegenüber Alternativen:**

- SKILL.md bleibt Single Source of Truth für beide Welten (Claude Code CLI + Agent Garage)
- Keine Prompt-Inflation im Phase-Agent
- Neue Skills einfach als neue Markdown-Datei hinzufügen — kein neuer n8n-Workflow nötig

**Implementierungsaufwand:** Gering — ein Custom Code Tool, keine weitere Infrastruktur.

### 3. Qdrant / Vektor-DB nicht genutzt

Qdrant läuft, wird aber in keinem Workflow genutzt. Dabei ist es die Grundlage für:

- Codebase-Suche (Code-Companion kann Codebase durchsuchen)
- Anforderungs-RAG (bestehende Requirements als Kontext)
- Runbook-RAG (Incident Response mit Wissen aus vergangenen Vorfällen)

**Implementierungsaufwand:** Mittel — n8n Qdrant-Nodes existieren, Indexierungs-Pipeline fehlt.

### 4. Cross-Session Memory

Heute: jede Chat-Session ist isoliert.
Gebraucht: Entscheidungen aus Requirements-Phase fließen in Testing-Phase.

**Implementierungsaufwand:** Mittel — Qdrant als persistenter Memory Store konfigurieren.

### 5. GitHub MCP Server

GitHub Actions (Claude Code Review) läuft, ist aber vom Chat entkoppelt. Für das vollständige Erlebnis braucht der Nutzer:

- PR-Status im Chat sehen
- PR aus Chat erstellen lassen
- Branch-Erstellung aus Chat

**Implementierungsaufwand:** Gering-Mittel — GitHub REST API wrappen als MCP Server.

### 6. Fehlende Open WebUI Personas

Heute: 2 Personas (User Story Creator, SDLC Agents).
Gebraucht: Personas für alle 5 SDLC-Phasen + SDLC Supervisor.

**Implementierungsaufwand:** Gering — Open WebUI Persona-Konfiguration.

### 7. LLM Router (Ollama vs Claude API)

Heute: Alles läuft auf Ollama.
Gebraucht: Komplexe Aufgaben (Code Review, Architektur) → Claude API; einfache (Klassifikation, Summary) → Ollama.

**Implementierungsaufwand:** Mittel — Routing-Logic in n8n-Workflows.

---

## Priorisierte Lückenliste

### 🔴 P0 — Für Demo unbedingt nötig

| #   | Was                                                                    | Warum                                                       | Aufwand          |
| --- | ---------------------------------------------------------------------- | ----------------------------------------------------------- | ---------------- |
| 1   | **SDLC Supervisor Workflow (n8n)**                                     | Kernstück der Integration — koordiniert alle Phasen         | M                |
| 2   | **Strukturierter Klärungsdialog** (Elicitation Agent als n8n-Workflow) | Demo-Step 1+2: BA gibt Feature ein, Agent stellt Rückfragen | S                |
| 3   | **Jira MCP vollständig** (inkl. Ticket lesen + Epic-Kontext)           | Demo-Step 6: Ticket-Fetch vor Implementierung               | S (Stub → Impl.) |
| 4   | **GitHub MCP Server**                                                  | Demo-Step 10+11: PR erstellen + AI Review im Chat sichtbar  | M                |
| 5   | **Open WebUI Personas** für alle 5 Phasen + Supervisor                 | Benutzer-Einstiegspunkt                                     | XS               |

### 🟠 P1 — Für vollständiges SDLC-Erlebnis nötig

| #   | Was                                                               | Warum                                                                 | Aufwand       |
| --- | ----------------------------------------------------------------- | --------------------------------------------------------------------- | ------------- |
| 6   | **Qdrant als Knowledge Base** (Codebase + Requirements indexiert) | Code-Companion braucht Codebase-Kontext                               | M             |
| 7   | **CI/CD MCP Server** (GitHub Actions API)                         | Pipeline-Status im Chat, Test-Trigger                                 | M             |
| 8   | **Confluence MCP** (Stub → Implementierung)                       | Requirements-Dokumentation, Runbooks                                  | S             |
| 9   | **SDLC Supervisor mit Cross-Phase Memory** (Qdrant)               | Kontext-Kontinuität über Phasen hinweg                                | M             |
| 10  | **Skill-Implementierungen P1**                                    | generate-user-stories, review-pr, generate-tests, root-cause-analysis | S-M pro Skill |

### 🟡 P2 — Für Enterprise-Reifegrad

| #   | Was                                  | Warum                                                     | Aufwand     |
| --- | ------------------------------------ | --------------------------------------------------------- | ----------- |
| 11  | **Prometheus/Grafana MCP**           | Observability im Chat                                     | M           |
| 12  | **ArgoCD/Kubernetes MCP**            | Deployment-Control                                        | M           |
| 13  | **LLM Router** (Ollama ↔ Claude API) | Kostenoptimierung, Qualität                               | M           |
| 14  | **SonarQube MCP**                    | Code-Qualität im Chat                                     | S           |
| 15  | **LaunchDarkly/Unleash MCP**         | Feature Flag Management                                   | S           |
| 16  | **Skill-Implementierungen P2**       | changelog, release-notes, incident-summary, rollback-plan | S pro Skill |

### 🟢 P3 — Vollständigkeit

| #   | Was                                         | Warum                                 | Aufwand |
| --- | ------------------------------------------- | ------------------------------------- | ------- |
| 17  | **Slack MCP**                               | Incident-Kommunikation, Announcements | S       |
| 18  | **PagerDuty/OpsGenie MCP**                  | Incident Management                   | S       |
| 19  | **Phase 2 Plugin** (Business Alignment)     | Business Case, ROI                    | L       |
| 20  | **Phase 16 Plugin** (Compliance/Governance) | GDPR, SOC2, SBOM                      | L       |
| 21  | **Phase 17 Plugin** (Knowledge Sharing)     | Retros, Onboarding                    | L       |
| 22  | **Figma MCP**                               | UX/UI Design-Phase                    | M       |

---

## Zusammenfassung: Die 5 wichtigsten Bausteine

```
1. SDLC Supervisor (n8n)
   ┌──────────────────────────────────────────────────┐
   │  Supervisor kennt alle Phasen, alle Skills,      │
   │  delegiert wie Manager-Agent — aber auf 5x mehr  │
   │  Fähigkeiten. Basiert auf bestehendem Pattern.   │
   └──────────────────────────────────────────────────┘

2. GitHub MCP Server
   ┌──────────────────────────────────────────────────┐
   │  GitHub REST API als MCP Server. PRs erstellen,  │
   │  Status lesen, Code Review triggern. Verbindet   │
   │  GitHub Actions Claude Review mit dem Chat.      │
   └──────────────────────────────────────────────────┘

3. Elicitation Workflow (strukturierter Dialog)
   ┌──────────────────────────────────────────────────┐
   │  Multi-Turn n8n Workflow: User gibt Feature-Idee │
   │  ein, Agent stellt Klärungsfragen bis vollständige│
   │  Anforderung vorliegt → dann Jira-Push.          │
   └──────────────────────────────────────────────────┘

4. Qdrant als aktiver Knowledge Store
   ┌──────────────────────────────────────────────────┐
   │  Codebase indexieren → Code-Companion hat        │
   │  Codebase-Kontext. Requirements indexieren →     │
   │  Cross-Phase Memory. Läuft bereits, ungenutzt.   │
   └──────────────────────────────────────────────────┘

5. `load_skill` Custom Code Tool
   ┌──────────────────────────────────────────────────┐
   │  Ein einziges Tool pro Phase-Agent das SKILL.md  │
   │  zur Laufzeit aus dem Repo liest. Skills bleiben │
   │  Markdown — kein separater Sub-Workflow pro      │
   │  Skill. Single Source of Truth für CLI + Garage. │
   └──────────────────────────────────────────────────┘
```

---

## Was NICHT gebaut werden muss

- **Phase 5 (Sprint Planning):** Jira-Agent deckt Ticket-Zuweisung ab. Rest ist Jira-native.
- **Phase 13 (Go-Live Kommunikation):** Marketing-Aufgabe außerhalb Scope.
- **Phase 16/17 (Compliance, Knowledge Sharing):** P3 — erst wenn der Kern läuft.
- **Lokale Dev-Environments:** Developer-seitig, kein Copilot-Feature.
- **Pair/Mob Programming:** Geschieht in IDE — Copilot gibt Kontext, macht keinen Screen-Share.
- **Multi-Browser-Testing, UAT:** Außerhalb automatisierbarem Scope.

---

_Quelldokumente: `/workspace/twodigits-marketplace/sdlc/integration-agent-garage.md`, `/workspace/twodigits-marketplace/sdlc/sdlc_tasks.csv`, `/workspace/agent-garage/README.md`, `/workspace/agent-garage/docker-compose.yml`, `/workspace/agent-garage/n8n/backup/workflows/_`\*
