---
name: deep-analysis-design-level
description: >
  Prowadzi fazę Design Level Event Stormingu. Uruchamiaj jako Skill 3 gdy orkiestrator
  zleca analizę Design Level. Wymaga ukończonych Fazy 1 i Fazy 2.
  Modeluje agregaty, encje, value objects i kontrakty per bounded context.
  Zapisuje wynik do state/phase-3-output.md.
tools: Read, Write, Bash, LS
---

# Deep Analysis — Design Level (Skill 3)

## STATUS: SZKIELET — do rozbudowy

Aktualnie zaimplementowane: struktura, dane wejściowe, kroki ogólne, handoff.
Szczegółowa logika modelowania agregatów i value objects: TODO.

---

## Dane wejściowe od orkiestratora

```
- tryb pracy (1 / 2 / 3 / 4)
- state/inputs/processed/phase-3-seed.md (jeśli istnieje)
- state/phase-1-output.md (wymagany)
- state/phase-2-output.md (wymagany)
- lista bounded contexts do analizy (podzbiór lub wszystkie)
- ścieżka do katalogu state/
```

---

## Krok 0 — Utwórz strukturę katalogów

```
state/
  phase-3/
    working/
      aggregates/     ← jeden plik per agregat
      value-objects/
      contracts/
```

---

## Krok 1 — Załaduj kontekst

Wczytaj `state/phase-2-output.md`. Wyświetl listę BC do wyboru.
Ustal kolejność pracy: zacznij od Core Domain.

---

## Krok 2 — Modelowanie agregatów (TODO)

> Do zaimplementowania: dla każdego BC z phase-2-output.md
> - identyfikacja agregat root
> - właściwości i niezmienniki
> - metody / komendy

---

## Krok 3 — Value Objects i Encje (TODO)

> Do zaimplementowania: rozróżnienie między encją (tożsamość) a value object (wartość)

---

## Krok 4 — Kontrakty między kontekstami (TODO)

> Do zaimplementowania: zdarzenia integracyjne, DTO, interfejsy
> na podstawie context-map z phase-2-output.md

---

## Krok 5 — Synteza i zapis wyniku

Wczytaj `templates/phase-3-template.md`.
Zapisz do `state/phase-3-output.md`.

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "design-level",
  "status": "completed",
  "output_file": "state/phase-3-output.md",
  "open_questions": [],
  "suggested_next": "specialist | end",
  "user_decision_required": true
}
```
