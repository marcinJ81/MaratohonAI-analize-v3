---
phase: big-picture
session: [nazwa projektu]
generated: [data]
status: completed | partial
coverage: [X%]
sources:
  - state/inputs/processed/phase-1-seed.md
  - user-input
---

# Big Picture — [Nazwa domeny]

## Kontekst
[2-3 zdania: co analizowano, jaki był cel, ocena złożoności systemu]

---

## Aktorzy

| ID    | Nazwa | Rola | Zewnętrzny? | Uwagi |
|-------|-------|------|-------------|-------|
| A-001 |       |      | tak / nie   |       |

---

## Zdarzenia domenowe

Zdarzenia zapisuj w czasie przeszłym (np. "Zamówienie złożone", "Płatność zatwierdzona").

| ID    | Zdarzenie | Aktor (ID) | Obszar | Hot Spot? |
|-------|-----------|------------|--------|-----------|
| E-001 |           |            |        | tak / nie |

---

## Granice systemu

### Opis granic
[Gdzie kończy się system, co jest poza nim]

### Integracje zewnętrzne
| ID    | System zewnętrzny | Kierunek          | Typ integracji | Uwagi |
|-------|-------------------|-------------------|----------------|-------|
| I-001 |                   | wchodzi / wychodzi / obustronny | sync / async / batch | |

---

## Hot Spoty

Hot Spot = miejsce konfliktu, niejasności, ryzyka lub brakującej wiedzy domenowej.

| ID    | Opis | Powód                                    | Powiązane zdarzenia (ID) | Priorytet         |
|-------|------|------------------------------------------|--------------------------|-------------------|
| H-001 |      | konflikt / niejasność / ryzyko / brak wiedzy |                      | wysoki / średni / niski |

---

## Wnioski

| ID    | Wniosek | Pewność                  | Źródło                   |
|-------|---------|--------------------------|--------------------------|
| W-001 |         | wysoka / średnia / niska | user-input / derived     |

---

## Otwarte pytania

| ID    | Pytanie | Priorytet              | Dotyczy  | Przeniesione do fazy |
|-------|---------|------------------------|----------|----------------------|
| Q-001 |         | blocker / nice-to-have | [obszar] | process-level / —    |

---

## Korekty i uzupełnienia (retrospektywne)

<!-- Sekcja wypełniana przez orkiestratora gdy późniejsza faza zmienia wnioski tej fazy. -->
<!-- Format każdej korekty:                                                               -->
<!--                                                                                      -->
<!-- [kontekst]: Wniosek [ID] wymaga korekty po Fazie [N].                               -->
<!-- Poprzednio: [stary wniosek]                                                          -->
<!-- Aktualnie:  [nowy wniosek]                                                           -->
<!-- Powód:      [co zmieniło ocenę]                                                      -->
