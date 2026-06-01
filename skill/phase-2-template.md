---
phase: process-level
session: [nazwa projektu]
generated: [data]
status: completed | partial
coverage: [X%]
sources:
  - state/inputs/processed/phase-2-seed.md
  - state/phase-1-output.md
  - user-input
---

# Process Level — [Nazwa domeny]

## Kontekst
[2-3 zdania: co analizowano, które bounded contexts objęto, skąd pochodzi wiedza]

---

## Core Domain

**Nazwa:** [nazwa]
**Uzasadnienie:** [dlaczego to jest core — co stanowi przewagę konkurencyjną lub kluczową wartość]

---

## Bounded Contexts

| ID    | Nazwa | Odpowiedzialność | Typ                              | Właściciel / Zespół |
|-------|-------|-----------------|----------------------------------|---------------------|
| BC-001 |      |                 | Core / Supporting / Generic      |                     |

---

## Przepływy procesów

<!-- Jeden blok per proces. Kopiuj blok dla każdego kolejnego przepływu. -->

### [Nazwa procesu] — [BC-ID]

**Wyzwalacz:** [co uruchamia proces — zdarzenie, aktor, harmonogram]
**Rezultat:** [co jest wynikiem poprawnego przebiegu]

**Kroki:**
1. [krok]
2. [krok]

**Reguły biznesowe:**
| ID     | Reguła | Egzekwowana w kroku |
|--------|--------|---------------------|
| RB-001 |        |                     |

**Wyjątki i ścieżki alternatywne:**
| ID    | Warunek | Akcja | Skutek |
|-------|---------|-------|--------|
| EX-001 |        |       |        |

---

## Zależności między kontekstami

| ID    | Z (BC-ID) | Do (BC-ID) | Typ relacji                                    | Kierunek                  | Uwagi |
|-------|-----------|------------|------------------------------------------------|---------------------------|-------|
| D-001 |           |            | upstream-downstream / shared-kernel / ACL / OHS | synchroniczny / asynchroniczny | |

---

## Polityki (Policies)

Polityka = reguła która łączy zdarzenie z akcją ("gdy X, wtedy Y").

| ID    | Zdarzenie wyzwalające (E-ID) | Polityka / Reguła | Akcja / Komenda | BC (ID) |
|-------|------------------------------|-------------------|-----------------|---------|
| P-001 |                              |                   |                 |         |

---

## Wnioski

| ID    | Wniosek | Pewność                  | Źródło                   |
|-------|---------|--------------------------|--------------------------|
| W-001 |         | wysoka / średnia / niska | user-input / derived     |

---

## Otwarte pytania

| ID    | Pytanie | Priorytet              | Dotyczy  | Przeniesione do fazy |
|-------|---------|------------------------|----------|----------------------|
| Q-001 |         | blocker / nice-to-have | [obszar] | design-level / —     |

---

## Korekty i uzupełnienia (retrospektywne)

<!-- Sekcja wypełniana przez orkiestratora gdy późniejsza faza zmienia wnioski tej fazy. -->
<!-- Format każdej korekty:                                                               -->
<!--                                                                                      -->
<!-- [kontekst]: Wniosek [ID] wymaga korekty po Fazie [N].                               -->
<!-- Poprzednio: [stary wniosek]                                                          -->
<!-- Aktualnie:  [nowy wniosek]                                                           -->
<!-- Powód:      [co zmieniło ocenę]                                                      -->
