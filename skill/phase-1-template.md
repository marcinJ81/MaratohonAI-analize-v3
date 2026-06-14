---
phase: big-picture
session: [nazwa projektu]
generated: [data]
status: completed | partial
coverage: [X%]
input-quality: accepted | n/a
lossy-files: [N]
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

| ID    | Zdarzenie | Aktor (ID) | Typ                              | Obszar | Hot Spot? |
|-------|-----------|------------|----------------------------------|--------|-----------|
| E-001 |           |            | domenowe / zewnętrzne / czasowe  |        | tak / nie |

---

## Pivotal Events

Pivotal event = zdarzenie po którym stan systemu zmienia się fundamentalnie.
Prefiks `PE-` jest globalny — Faza 2 kontynuuje numerację i uzupełnia kolumnę granic BC.

| ID     | Zdarzenie (E-ID) | Dlaczego pivotal                                          |
|--------|------------------|-----------------------------------------------------------|
| PE-001 |                  | punkt bez powrotu / zmiana fazy procesu / zmiana właściciela |

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
Nie usuwaj rozwiązanych — oznacz `resolved` z powodem (track decyzji).

| ID    | Opis | Powód                                    | Powiązane zdarzenia (ID) | Status                     | Priorytet                       |
|-------|------|------------------------------------------|--------------------------|----------------------------|---------------------------------|
| H-001 |      | konflikt / niejasność / ryzyko / brak wiedzy |                      | open / resolved: [powód]   | blocker / wysoki / średni / niski |

---

## Read Modele (opcjonalnie — jeśli bleed-through)

<!-- Wypełniane tylko gdy użytkownik wybrał bleed-through read modeli w Kroku 0. -->

| ID     | Moment (E-ID / PE-ID) | Kto musi widzieć | Co musi być widoczne |
|--------|------------------------|------------------|----------------------|
| RM-001 |                        |                  |                      |

---

## Zasady biznesowe (opcjonalnie — jeśli bleed-through)

<!-- Wypełniane tylko gdy użytkownik wybrał bleed-through zasad biznesowych w Kroku 0. -->
<!-- Zasady na poziomie przepływu, bez szczegółów implementacji.                       -->

| ID     | Reguła | Wpływa na (E-ID) |
|--------|--------|-------------------|
| RB-001 |        |                   |

---

## Wnioski

<!-- Reguła pewności: wniosek oparty wyłącznie na materiale lossy → maksymalnie "średnia". -->

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
