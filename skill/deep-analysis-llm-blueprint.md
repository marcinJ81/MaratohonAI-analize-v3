---
name: deep-analysis-llm-blueprint
description: >
  Generuje strukturalny blueprint w formacie YAML na podstawie wyników faz analizy domenowej.
  Output jest zdatny do konsumpcji przez LLM w celu generowania kodu (C#, interfejsy, szkielet projektu).
  Uruchamiaj gdy użytkownik wybrał opcję LLM Blueprint (opcja 3 lub 4).
  Wymaga ukończonej co najmniej fazy Process Level.
tools: Read, Write, LS
---

# Deep Analysis — LLM Blueprint Generator (Skill LB)

## STATUS: SZKIELET — do rozbudowy

Aktualnie zaimplementowane: struktura katalogów, schemat YAML, wczytanie danych, stub outputu.
Logika ekstrakcji i mapowania danych z faz na YAML: TODO.

---

## Dane wejściowe od orkiestratora

```
- lista ukończonych faz
- ścieżki do phase-X-output.md
- ścieżka do katalogu state/
```

Minimalne wymaganie: `state/phase-2-output.md` (Process Level) musi istnieć.

---

## Krok 0 — Utwórz strukturę katalogów

```
state/output/
  blueprint/
```

---

## Krok 1 — Wczytaj dane źródłowe

Wczytaj wszystkie dostępne pliki output:

```
state/phase-1-output.md  ← aktorzy, zdarzenia, granice (jeśli istnieje)
state/phase-2-output.md  ← bounded contexts, core domain, przepływy (wymagany)
state/phase-3-output.md  ← agregaty, encje, kontrakty (jeśli istnieje)
```

---

## Krok 2 — Mapuj dane na schemat YAML

> TODO: implementacja ekstrakcji i mapowania

Docelowy schemat blueprint:

```yaml
# state/output/blueprint/solution-blueprint.yaml
---
meta:
  session: [nazwa]
  generated: [data]
  sources:
    - state/phase-1-output.md
    - state/phase-2-output.md
  completeness: partial | full

domain:
  name: [nazwa domeny]
  core_domain: [nazwa core domain]
  ubiquitous_language:
    - term: [termin]
      definition: [definicja]

bounded_contexts:
  - id: BC-001
    name: [nazwa]
    type: core | supporting | generic
    aggregates:
      - id: AGG-001
        name: [nazwa]
        is_root: true | false
        properties:
          - name: [nazwa]
            type: string | int | bool | [typ domenowy]
        invariants:
          - [reguła biznesowa jako zdanie]
        methods:
          - name: [nazwa metody / komendy]
            description: [co robi]
    events:
      - id: E-001
        name: [NazwaWCzasie Przeszłym]
        trigger: [co wywołuje]
        payload:
          - name: [nazwa]
            type: [typ]
    commands:
      - id: CMD-001
        name: [NazwaImperatyw]
        handler: [AGG-ID lub serwis]
    repositories:
      - id: REP-001
        aggregate: AGG-001
        interface_name: I[Nazwa]Repository
        methods:
          - GetById
          - Save

context_map:
  - from: BC-001
    to: BC-002
    relation: upstream-downstream | ACL | shared-kernel | OHS
    direction: sync | async | batch
    contract: [nazwa zdarzenia integracyjnego lub DTO]

open_items:
  - id: Q-001
    type: blocker | decision | assumption
    description: [opis]
    affects: [BC-ID lub AGG-ID]
```

---

## Krok 3 — Zapisz output

```
state/output/blueprint/solution-blueprint.yaml
```

Dodatkowo wygeneruj skrócone podsumowanie w MD dla człowieka:

```
state/output/blueprint/solution-blueprint-summary.md
```

Format podsumowania:
```markdown
---
generated-by: llm-blueprint
session: [nazwa]
---

# Blueprint Summary — [nazwa domeny]

## Core Domain
[nazwa + 1 zdanie uzasadnienia]

## Bounded Contexts ([liczba])
| ID | Nazwa | Typ | Agregaty |
|---|---|---|---|

## Context Map
| Z | Do | Relacja |
|---|---|---|

## Open Items — Blokery ([liczba])
| ID | Opis | Dotyczy |
|---|---|---|
```

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "llm-blueprint",
  "status": "completed | partial | error",
  "output_files": {
    "yaml": "state/output/blueprint/solution-blueprint.yaml",
    "summary": "state/output/blueprint/solution-blueprint-summary.md"
  },
  "completeness": "partial | full",
  "open_questions": [],
  "suggested_next": "end",
  "user_decision_required": false
}
```

`status: partial` gdy brakuje fazy Design Level — blueprint zawiera BC i przepływy,
ale nie zawiera agregatów i kontraktów.
