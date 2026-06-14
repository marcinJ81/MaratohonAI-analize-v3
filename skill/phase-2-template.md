---
phase: process-level
session: [nazwa projektu]
generated: [data]
status: completed | partial
coverage: [X%]
input-quality: accepted | n/a
lossy-files: [N]
sources:
  - state/inputs/processed/phase-2-seed.md
  - state/phase-1-output.md
  - user-input
---

# Process Level — [Nazwa domeny]

## Kontekst
[2-3 zdania: co analizowano, które obszary objęto, skąd pochodzi wiedza]

---

## Subdomeny

<!-- Typologia dotyczy subdomen (przestrzeń problemu), nie bounded contexts. -->

| ID     | Nazwa | Odpowiedzialność | Typ                         | Testy rozstrzygające (T1–T4) | Zdarzenia z Fazy 1 |
|--------|-------|------------------|-----------------------------|------------------------------|--------------------|
| SD-001 |       |                  | Core / Supporting / Generic | [np. T1: brak gotowego, T4: najdroższy błąd] | [E-IDs] |

---

## Core Domain

**Nazwa:** [nazwa subdomeny, SD-ID]
**Uzasadnienie:** [dlaczego to jest core — co stanowi przewagę konkurencyjną, które testy zdecydowały]

---

## Pivotal Events

<!-- Zdarzenia nieodwracalne lub rozdzielające fazy procesu — kandydaci na granice BC.  -->
<!-- Numeracja PE- kontynuowana z Fazy 1. PE zdegradowane oznaczaj w kolumnie Status — nie usuwaj. -->

| ID     | Zdarzenie (E-ID) | Dlaczego pivotal                              | Źródło            | Status                          | Granica między (BC-ID → BC-ID) |
|--------|------------------|-----------------------------------------------|-------------------|---------------------------------|--------------------------------|
| PE-001 |                  | punkt bez powrotu / zmiana fazy / zmiana właściciela / widoczne na zewnątrz | Faza 1 / Faza 2 | aktualny / zdegradowany: [powód] | |

---

## Bounded Contexts

<!-- Punkt startu: 1 subdomena = 1 BC. Odstępstwa zapisuj w kolumnie Uwagi z uzasadnieniem. -->

| ID     | Nazwa | Odpowiedzialność | Subdomena źródłowa (SD-ID) | Typ (z subdomeny) | Właściciel / Zespół | Uwagi |
|--------|-------|------------------|----------------------------|-------------------|---------------------|-------|
| BC-001 |       |                  |                            | Core / Supporting / Generic | | |

---

## Przepływy procesów

<!-- Jeden blok per proces. Kopiuj blok dla każdego kolejnego przepływu. -->
<!-- Gramatyka kroku: [Aktor] --[Komenda]--> [BC/System] ==> [Zdarzenie]   -->

### [P-ID] [Nazwa procesu] — [BC-ID]

**Wyzwalacz:** [aktor / harmonogram / zdarzenie z innego BC / system zewnętrzny]
**Rezultat:** [co jest wynikiem poprawnego przebiegu — zdarzenie końcowe]

**Kroki:**
1. [Aktor] --[Komenda]--> [BC/System] ==> [Zdarzenie]
2. ...

**Read modele:**
| Krok | Aktor | Co musi widzieć, żeby podjąć decyzję |
|------|-------|--------------------------------------|
|      |       |                                      |

**Reguły biznesowe:**
| ID     | Reguła | Egzekwowana w kroku |
|--------|--------|---------------------|
| RB-001 |        |                     |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek | Akcja | Skutek |
|--------|---------|-------|--------|
| EX-001 |         |       |        |

**Przepływ odwrotny (błędy, kompensacje, cofnięcia):**
- [co się dzieje, gdy proces trzeba cofnąć lub skompensować]

**Weryfikacja odwrotną narracją:** [tak / nie — luki znalezione: ...]

---

## Polityki (Policies)

<!-- Polityka = reguła łącząca zdarzenie z akcją: "gdy X, wtedy Y".                 -->
<!-- Typ "ludzka" = procedura, nawyk, czeklista — ukryta logika, kandydat na hot spot. -->

| ID      | Zdarzenie wyzwalające (E-ID) | Polityka / Reguła | Komenda / Akcja | BC (ID) | Typ                  | Przekracza granicę BC? |
|---------|------------------------------|-------------------|-----------------|---------|----------------------|------------------------|
| POL-001 |                              |                   |                 |         | automatyczna / ludzka | tak (→ D-ID) / nie    |

---

## Relacje między kontekstami

<!-- Wzorce (Evans, Vernon): Partnership / Shared Kernel / Customer–Supplier /      -->
<!-- Conformist / ACL / OHS / Published Language / Separate Ways / Big Ball of Mud. -->

| ID    | Upstream (BC-ID) | Downstream (BC-ID) | Wzorzec | Heurystyka (R1–R5) | Komunikacja                    | Co jest przekazywane              | Uwagi |
|-------|------------------|--------------------|---------|--------------------|--------------------------------|-----------------------------------|-------|
| D-001 |                  |                    |         |                    | synchroniczna / asynchroniczna | zdarzenie integracyjne / DTO / API |       |

---

## Hot spoty

<!-- Nowe hot spoty tej fazy + rozstrzygnięcia hot spotów z Fazy 1. -->

| ID      | Opis | Źródło (krok / proces) | Status            | Rozstrzyga HS z Fazy 1 (H-ID) |
|---------|------|------------------------|-------------------|--------------------------------|
| HS2-001 |      |                        | open / resolved   | —                              |

---

## Wnioski

<!-- Reguła pewności: wniosek oparty wyłącznie na materiale lossy → maksymalnie "średnia". -->

| ID    | Wniosek | Pewność                  | Źródło               |
|-------|---------|--------------------------|----------------------|
| W-001 |         | wysoka / średnia / niska | user-input / derived |

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
