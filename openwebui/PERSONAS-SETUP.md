# Open WebUI Personas — Setup Guide

> Stand: 2026-02-20
> Voraussetzung: n8n läuft, SDLC Supervisor Workflow ist importiert und aktiviert.

---

## Wie die Integration funktioniert

Jede Persona in Open WebUI ist ein **Modell**, das über eine Python-Funktion ("n8n-pipe") Nachrichten an einen n8n-Webhook weiterleitet. Das Muster:

```
User tippt in Open WebUI
  → n8n-pipe Funktion (Python, in Open WebUI installiert)
  → HTTP POST an n8n Webhook
  → n8n Workflow verarbeitet Anfrage
  → Antwort zurück an Open WebUI
```

---

## Schritt 1: n8n-pipe Funktion prüfen

Die n8n-pipe Funktion ist wahrscheinlich bereits für die bestehenden Personas installiert.

**Prüfen:** Open WebUI → Admin Panel → Functions

Falls keine `n8n-pipe` Funktion vorhanden:
1. Admin Panel → Functions → "+" (neu erstellen)
2. Name: `n8n-pipe`
3. Code: siehe unten

```python
"""
title: n8n-pipe
author: agent-garage
version: 1.1.0
"""
from typing import Optional, Callable, Awaitable
from pydantic import BaseModel, Field
import time
import requests


class Pipe:
    class Valves(BaseModel):
        n8n_url: str = Field(default="http://n8n:5678/webhook/sdlc-supervisor")
        n8n_bearer_token: str = Field(default="")
        input_field: str = Field(default="chatInput")
        response_field: str = Field(default="output")
        emit_interval: float = Field(
            default=2.0, description="Interval in seconds between status emissions"
        )
        enable_status_indicator: bool = Field(
            default=True, description="Enable or disable status indicator emissions"
        )

    def __init__(self):
        self.type = "pipe"
        self.id = "n8n_pipe"
        self.name = "N8N Pipe"
        self.valves = self.Valves()
        self.last_emit_time = 0

    async def emit_status(
        self,
        __event_emitter__: Callable[[dict], Awaitable[None]],
        level: str,
        message: str,
        done: bool,
    ):
        current_time = time.time()
        if (
            __event_emitter__
            and self.valves.enable_status_indicator
            and (
                current_time - self.last_emit_time >= self.valves.emit_interval or done
            )
        ):
            await __event_emitter__(
                {
                    "type": "status",
                    "data": {
                        "status": "complete" if done else "in_progress",
                        "level": level,
                        "description": message,
                        "done": done,
                    },
                }
            )
            self.last_emit_time = current_time

    async def pipe(
        self,
        body: dict,
        __user__: Optional[dict] = None,
        __event_emitter__: Callable[[dict], Awaitable[None]] = None,
        __event_call__: Callable[[dict], Awaitable[dict]] = None,
    ) -> None:
        await self.emit_status(__event_emitter__, "info", "Calling N8N Workflow...", False)

        messages = body.get("messages", [])

        if not messages:
            await self.emit_status(__event_emitter__, "error", "No messages found in the request body", True)
            await __event_emitter__(
                {
                    "type": "message",
                    "data": {"role": "assistant", "content": "No messages found in the request body"},
                }
            )
            return None

        question = messages[-1]["content"]
        if "Prompt: " in question:
            question = question.split("Prompt: ")[-1]

        try:
            headers = {
                "Content-Type": "application/json",
            }
            if self.valves.n8n_bearer_token:
                headers["Authorization"] = f"Bearer {self.valves.n8n_bearer_token}"

            payload = {
                "sessionId": f"{__user__['id']} - {messages[0]['content'].split('Prompt: ')[-1][:100]}",
                self.valves.input_field: question,
            }

            response = requests.post(
                self.valves.n8n_url,
                json=payload,
                headers=headers,
                timeout=120,
            )
            response.raise_for_status()
            n8n_response = response.json()[self.valves.response_field]

            await __event_emitter__(
                {
                    "type": "message",
                    "data": {"role": "assistant", "content": n8n_response},
                }
            )
            await self.emit_status(__event_emitter__, "info", "Success", True)

        except Exception as e:
            await self.emit_status(
                __event_emitter__,
                "error",
                f"Error during sequence execution: {str(e)}",
                True,
            )
            await __event_emitter__(
                {
                    "type": "message",
                    "data": {"role": "assistant", "content": f"Error: {str(e)}"},
                }
            )

        return None
```

---

## Schritt 2: SDLC Supervisor Persona anlegen

**Open WebUI → Admin Panel → Models → "+" (neu)**

### Persona 1: SDLC Supervisor

| Feld | Wert |
|------|------|
| **Model ID** | `sdlc-supervisor` |
| **Name** | `SDLC Supervisor` |
| **Base Model** | `n8n-pipe` (die Funktion aus Schritt 1) |
| **Description** | Koordiniert alle SDLC-Phasen: Requirements, Entwicklung, Testing, Release und Operations. Lädt spezialisierte Skills aus dem Marketplace. |
| **System Prompt** | *(leer lassen — Supervisor hat eigenen System-Prompt im n8n-Workflow)* |
| **Tags** | `sdlc`, `supervisor` |

**Funktion konfigurieren (Valves):**
| Valve | Wert |
|-------|------|
| `n8n_url` | `http://n8n:5678/webhook/sdlc-supervisor` |
| `input_field` | `chatInput` |
| `response_field` | `output` |

---

## Schritt 3: Phase-Personas anlegen (P0 — alle routen zum Supervisor)

Alle 5 Phase-Personas nutzen **dieselbe n8n-pipe** und **denselben Webhook** wie der Supervisor.
Der Unterschied liegt im System Prompt: er gibt der KI einen Phasen-Fokus.

### Persona 2: Requirements Engineer

| Feld | Wert |
|------|------|
| **Model ID** | `requirements-engineer` |
| **Name** | `Requirements Engineer` |
| **Base Model** | `n8n-pipe` |
| **Description** | Spezialist für Anforderungsanalyse: User Stories, Akzeptanzkriterien, Stakeholder-Analyse, Konflikt-Erkennung und Anforderungs-Extraktion. |
| **System Prompt** | Du bist ein Requirements Engineer. Fokussiere dich auf Anforderungsanalyse, User Stories, Akzeptanzkriterien und Anforderungsdokumentation. |

**Valves:** identisch zu SDLC Supervisor (`http://n8n:5678/webhook/sdlc-supervisor`)

---

### Persona 3: Code Companion

| Feld | Wert |
|------|------|
| **Model ID** | `code-companion` |
| **Name** | `Code Companion` |
| **Base Model** | `n8n-pipe` |
| **Description** | Unterstützt bei Implementierung, Code-Review, Debugging, Refactoring und Dokumentation. |
| **System Prompt** | Du bist ein Code Companion. Fokussiere dich auf Implementierung, Code-Qualität, Debugging, Refactoring und technische Dokumentation. |

**Valves:** `http://n8n:5678/webhook/sdlc-supervisor`

---

### Persona 4: Test Commander

| Feld | Wert |
|------|------|
| **Model ID** | `test-commander` |
| **Name** | `Test Commander` |
| **Base Model** | `n8n-pipe` |
| **Description** | Plant und generiert Tests: Unit Tests, E2E Tests, Testpläne, Coverage-Analyse und Performance-Tests. |
| **System Prompt** | Du bist ein Test Commander. Fokussiere dich auf Testplanung, Testgenerierung, Coverage-Analyse und Qualitätssicherung. |

**Valves:** `http://n8n:5678/webhook/sdlc-supervisor`

---

### Persona 5: Release Pilot

| Feld | Wert |
|------|------|
| **Model ID** | `release-pilot` |
| **Name** | `Release Pilot` |
| **Base Model** | `n8n-pipe` |
| **Description** | Koordiniert Releases: Changelogs, Release Notes, Deployment-Strategien, Rollback-Pläne und Feature Flags. |
| **System Prompt** | Du bist ein Release Pilot. Fokussiere dich auf Release-Vorbereitung, Deployment-Strategien, Changelogs und Release-Kommunikation. |

**Valves:** `http://n8n:5678/webhook/sdlc-supervisor`

---

### Persona 6: Ops Intelligence

| Feld | Wert |
|------|------|
| **Model ID** | `ops-intelligence` |
| **Name** | `Ops Intelligence` |
| **Base Model** | `n8n-pipe` |
| **Description** | SRE-Spezialist: Incident Response, Root Cause Analysis, Log-Analyse, SLO-Management und Anomalie-Erkennung. |
| **System Prompt** | Du bist ein Ops Intelligence Agent. Fokussiere dich auf Incident Response, Root Cause Analysis, Monitoring und SRE-Aufgaben. |

**Valves:** `http://n8n:5678/webhook/sdlc-supervisor`

---

## Hinweise zur Implementierung

### System Prompt Übergabe an n8n

Die aktuellen n8n-Workflows lesen nur `{{ $json.body.chatInput }}`. Der System Prompt aus Open WebUI wird noch nicht an den Supervisor übergeben. Das ist für P0 akzeptabel — der Supervisor hat seinen eigenen System-Prompt im n8n-Workflow.

Für eine spätere Iteration kann der System Prompt als `systemPrompt`-Feld im n8n-Pipe Payload mitgegeben und im Supervisor-Workflow verwendet werden.

### Warum `return None` statt einer Antwort-String?

Die Pipe sendet die Antwort direkt per `__event_emitter__` (`type: "message"`). Würde die `pipe()`-Methode zusätzlich einen String zurückgeben, erschiene die Antwort in OpenWebUI **doppelt**. `return None` signalisiert OpenWebUI: "Alles wurde bereits per Event gesendet, nichts mehr anhängen."

---

## Verifizierung

Nach dem Anlegen aller Personas:

1. Open WebUI → Neuer Chat → Modell: **SDLC Supervisor**
2. Testnachricht: `"Erstelle eine User Story für ein Login-Feature"`
3. Erwartetes Verhalten: Supervisor lädt `generate-user-stories` Skill, gibt strukturierte User Story zurück

Wenn der Chat antwortet: ✅ Integration funktioniert.
Wenn Timeout oder Fehler: n8n Webhook URL und Workflow-Aktivierung prüfen.
