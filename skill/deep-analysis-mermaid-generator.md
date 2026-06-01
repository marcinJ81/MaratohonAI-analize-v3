---
name: deep-analysis-mermaid-generator
description: >
  Generuje diagram Mermaid na podstawie wyników faz analizy domenowej.
  Uruchamiaj gdy użytkownik wybrał output w formie diagramu (opcja 1).
  Wymaga ukończonej co najmniej jednej fazy analizy.
  Obsługuje poziomy: Big Picture, Process Level, Design Level.
tools: Read, Write, LS
---

# Deep Analysis — Mermaid Generator (Skill MG)

## STATUS: SZKIELET — do rozbudowy

Aktualnie zaimplementowane: struktura katalogów, wczytanie danych, stub outputu.
Logika generowania diagramów: TODO.

---

## Dane wejściowe od orkiestratora

```
- lista ukończonych faz (BP / PL / DL)
- wybrany poziom diagramu: big-picture | process-level | design-level
- ścieżki do phase-X-output.md
- ścieżka do katalogu state/
```

---

## Krok 0 — Utwórz strukturę katalogów

```
state/output/
  mermaid/
```

---

## Krok 1 — Wczytaj dane źródłowe

Wczytaj pliki output dla wybranego poziomu:

| Poziom | Pliki źródłowe |
|---|---|
| Big Picture | `state/phase-1-output.md` |
| Process Level | `state/phase-1-output.md` + `state/phase-2-output.md` |
| Design Level | `state/phase-1-output.md` + `state/phase-2-output.md` + `state/phase-3-output.md` |

Jeśli wymagany plik nie istnieje — zwróć error z informacją która faza jest niekompletna.

---

## Krok 2 — Generuj diagram

> TODO: implementacja generowania Mermaid per poziom

**Big Picture — planowany format:**
```
graph LR — aktorzy, zdarzenia, granice systemu
```

**Process Level — planowany format:**
```
graph TD — bounded contexts, przepływy, zależności
```

**Design Level — planowany format:**
```
classDiagram — agregaty, encje, value objects
```

---

## Krok 3 — Zapisz output

```
state/output/mermaid/[poziom]-diagram.md
```

Format pliku:
```markdown
---
generated-by: mermaid-generator
level: big-picture | process-level | design-level
session: [nazwa]
sources: [lista plików źródłowych]
---

# Diagram — [poziom] — [nazwa domeny]

```mermaid
[wygenerowany diagram]
```
```

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "mermaid-generator",
  "status": "completed | error",
  "output_file": "state/output/mermaid/[poziom]-diagram.md",
  "open_questions": [],
  "suggested_next": "end",
  "user_decision_required": false
}
```
