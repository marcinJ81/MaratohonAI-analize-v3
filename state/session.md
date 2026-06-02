# Session: maraton-rowerowy
Data startu: 2026-06-01
Tryb: Tryb 2 — Ekspercki (użytkownik jako ekspert domenowy)
Złożoność systemu: średnia-wysoka

## Cel analizy
Przegląd procesu obsługi maratonu rowerowego oraz uzupełnienie brakujących części systemu. Użytkownik chce dojść do Process Level.

## Fazy
| Faza | Status | Plik outputu | Seed |
|---|---|---|---|
| Big Picture | completed | state/phase-1-output.md | state/inputs/processed/all-materials-processed.md |
| Process Level | completed | state/phase-2-output.md | state/inputs/processed/phase-2-seed.md |
| Design Level | not-started | state/phase-3-output.md | — |
| Specialist | not-started | state/phase-s-output.md | — |

## Materiały wejściowe
| Plik | Typ | Konwersja | Pokrycie BP | Pokrycie PL | Pokrycie DL |
|---|---|---|---|---|---|
| 18 × PNG (Event Storming) | obraz | lossy | 80% | 60% | 15% |

Szczegóły: state/inputs/processed/index.md

## Otwarte pytania (przeniesione do Fazy 3)
- Q-007 [blocker]: Sync lokalny→globalny — kolejkowanie czy merge przy długim offline?
- Q-008: Czy zawodnik widzi live ranking podczas zawodów?
- Q-009: Dane w dyplomie — automatycznie czy ręcznie?
- Q-010 [blocker]: Czy odwołania od kar wymagają śledzonego procesu w systemie?
- Q-011: Warunki promocji -100% — limity, uprawnienia?
- Q-012 [blocker]: Czy istnieją kategorie zawodnika (wiek/płeć) wpływające na ranking/nagrody?

## Decyzje użytkownika
- Tryb pracy: 2 (Ekspercki)
- Cel: Process Level (przegląd + uzupełnienie)
- Core domain: Pomiar i Wyniki

## Historia handoffów
- 2026-06-01 Faza 1 → completed (derived z materiałów), suggested_next: process-level
- 2026-06-01 Faza 2 → in-progress
- 2026-06-02 Faza 2 → completed (78% pokrycia), suggested_next: design-level, open_questions: Q-007, Q-010, Q-012 (blocker)
