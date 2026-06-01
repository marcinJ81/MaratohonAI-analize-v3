---
name: deep-analysis-specialist
description: >
  Prowadzi fazy specjalistyczne analizy domenowej (S1-S3).
  Uruchamiaj na żądanie użytkownika w dowolnym momencie sesji.
  Dostępne fazy: S1 (analiza ryzyka), S2 (integracje i API), S3 (migracja / legacy).
  Wymaga co najmniej jednej ukończonej fazy głównej.
tools: Read, Write, Bash, LS
---

# Deep Analysis — Specialist (Skill 4)

## STATUS: SZKIELET — do rozbudowy

Aktualnie zaimplementowane: struktura, definicje faz, handoff.
Logika per faza specjalistyczna: TODO.

---

## Dane wejściowe od orkiestratora

```
- która faza specjalistyczna: S1 | S2 | S3
- ścieżki do dostępnych phase-N-output.md
- ścieżka do katalogu state/
```

---

## Krok 0 — Utwórz strukturę katalogów

```
state/
  specialist/
    [S1 | S2 | S3]/
      working/
```

---

## Definicje faz specjalistycznych

### S1 — Analiza ryzyka

Wejście: phase-1-output.md + phase-2-output.md
Focus: hot spoty z BP, blokery z PL, ryzyka techniczne i domenowe
Output: `state/specialist/S1/risk-assessment.md`

> TODO: pytania i logika zbierania ryzyk

### S2 — Integracje i API

Wejście: phase-2-output.md (context-map)
Focus: kontrakty między BC, API zewnętrzne, zdarzenia integracyjne
Output: `state/specialist/S2/integration-map.md`

> TODO: modelowanie kontraktów API i event schema

### S3 — Migracja / Legacy

Wejście: wszystkie dostępne phase-N-output.md
Focus: mapowanie istniejącego systemu na nowy model domenowy, blast radius
Output: `state/specialist/S3/migration-plan.md`

> TODO: analiza gap między as-is a to-be

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "specialist",
  "specialist_phase": "S1 | S2 | S3",
  "status": "completed",
  "output_file": "state/specialist/[S]/[nazwa].md",
  "open_questions": [],
  "suggested_next": "end | specialist",
  "user_decision_required": true
}
```
