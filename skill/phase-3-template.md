---
phase: design-level
session: [nazwa projektu]
generated: [data]
status: completed | partial
coverage: [X%]
sources:
  - state/inputs/processed/phase-3-seed.md
  - state/phase-1-output.md
  - state/phase-2-output.md
---

# Design Level — [Nazwa domeny]

## Kontekst
[2-3 zdania: które BC analizowano, jaki był cel, co wniosła Faza 2]

---

## Agregaty per Bounded Context

<!-- Kopiuj blok per BC -->

### [Nazwa BC] (BC-ID)

#### Agregat: [Nazwa] (AGG-001)

**Agregat Root:** tak / nie
**Odpowiedzialność:** [jedno zdanie]

**Właściwości:**
| Nazwa | Typ | Niezmiennik |
|---|---|---|
| | | |

**Niezmienniki agregatu:**
- [reguła biznesowa której agregat pilnuje]

**Metody / Komendy:**
| ID | Nazwa | Wejście | Zdarzenie wynikowe |
|---|---|---|---|
| CMD-001 | | | E-ID |

---

## Value Objects

| ID | Nazwa | BC (ID) | Właściwości | Reguły walidacji |
|---|---|---|---|---|
| VO-001 | | BC-001 | | |

---

## Encje (nie będące agregat root)

| ID | Nazwa | BC (ID) | Tożsamość przez | Cykl życia |
|---|---|---|---|---|
| EN-001 | | BC-001 | | |

---

## Kontrakty między kontekstami

| ID | Z (BC-ID) | Do (BC-ID) | Typ | Nazwa kontraktu | Payload |
|---|---|---|---|---|---|
| CNT-001 | | | event / DTO / API | | |

---

## Repozytoria

| ID | Agregat (ID) | Nazwa interfejsu | Metody |
|---|---|---|---|
| REP-001 | AGG-001 | I[Nazwa]Repository | GetById, Save |

---

## Wnioski

| ID | Wniosek | Pewność | Źródło |
|---|---|---|---|
| W-001 | | wysoka / średnia / niska | user-input / derived |

---

## Otwarte pytania

| ID | Pytanie | Priorytet | Dotyczy | Przeniesione do fazy |
|---|---|---|---|---|
| Q-001 | | blocker / nice-to-have | [obszar] | specialist / — |

---

## Korekty i uzupełnienia (retrospektywne)

<!-- Sekcja wypełniana przez orkiestratora gdy późniejsza faza zmienia wnioski tej fazy. -->
<!-- Format: [kontekst]: Wniosek [ID] wymaga korekty po Fazie [N].                       -->
<!-- Poprzednio: [stary wniosek] / Aktualnie: [nowy wniosek] / Powód: [...]              -->
