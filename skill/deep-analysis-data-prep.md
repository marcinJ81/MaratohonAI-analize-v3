---
name: deep-analysis-data-prep
description: >
  Przetwarza materiały wejściowe dostarczone przez użytkownika przed uruchomieniem faz analizy.
  Uruchamiaj gdy użytkownik dostarczył jakiekolwiek materiały (obrazy, tekst, diagramy, ES).
  Wykonuje: składowanie oryginałów, konwersję, normalizację, ocenę złożoności systemu,
  ocenę pokrycia per faza, zasilanie wsteczne między fazami, budowanie seed files.
  Zwraca handoff z wynikiem pokrycia i ścieżkami seed files dla orkiestratora.
tools: Read, Write, Bash, Glob, LS
---

# Deep Analysis — Data Prep (Skill DP)

## Zasada nadrzędna

Ten skill przetwarza dane — nie analizuje domeny.
Każdy krok tworzy pliki na dysku przed przejściem do następnego.
Awaria w połowie = odtworzenie od ostatniego zapisanego kroku.

---

## Dane wejściowe od orkiestratora

```
- opis systemu (2-3 zdania z pytania orientacyjnego nr 3)
- ścieżki do materiałów dostarczonych przez użytkownika
- ścieżka do katalogu state/
- lista faz wybranych przez użytkownika (BP / PL / DL)
```

---

## Krok 0 — Utwórz strukturę katalogów

Przed jakimkolwiek przetwarzaniem utwórz katalogi dla tego kroku:

```
state/inputs/
  raw/
  processed/
```

Zasada ogólna: każdy skill tworzy własne katalogi na początku działania.
Nie zakładaj że katalog istnieje.

---

## Krok 1 — Ocena złożoności systemu

Na podstawie opisu systemu (opis z pytania 3) oceń złożoność:

```
Złożoność niska:   jeden kontekst, mało integracji, prosta domena
Złożoność średnia: kilka kontekstów lub integracji, reguły biznesowe
Złożoność wysoka:  wiele kontekstów, złożone reguły, dużo integracji
```

Zapisz wynik do `state/inputs/complexity.md`:

```markdown
---
generated-by: data-prep
---
# Ocena złożoności systemu

**Poziom:** niska / średnia / wysoka
**Uzasadnienie:** [2-3 zdania na podstawie opisu użytkownika]

## Opis systemu (źródło)
[dosłowny opis z pytania orientacyjnego]
```

---

## Krok 2 — Składowanie oryginałów

Skopiuj wszystkie materiały bez modyfikacji do `state/inputs/raw/`.
Zanotuj każdy plik w `state/inputs/processed/index.md` (tworzysz plik jeśli nie istnieje):

```markdown
# Inputs Index
| Plik | Typ | Konwersja | Użyty w fazach | Pokrycie BP | Pokrycie PL | Pokrycie DL |
|---|---|---|---|---|---|---|
```

---

## Krok 3 — Konwersja i normalizacja

Dla każdego pliku z `state/inputs/raw/`:

| Typ materiału | Akcja |
|---|---|
| Obraz / screenshot | Opisz zawartość. Wyekstrahuj: zdarzenia, aktorów, granice, hot spoty. Oznacz `converted: lossy` jeśli struktura mogła zostać utracona. |
| Tekst niestrukturalny | Znormalizuj do MD z sekcjami: Zdarzenia, Aktorzy, Granice, Hot Spoty, Pytania otwarte. |
| Gotowy ES / diagram | Wyekstrahuj dane per kategoria jak wyżej. |
| Plik MD / dokument | Sprawdź format. Jeśli zgodny — przepisz nagłówki. Jeśli nie — normalizuj jak tekst niestrukturalny. |

Zapisz każdy przetworzony plik do:
```
state/inputs/processed/[nazwa-pliku]-processed.md
```

Nagłówek każdego przetworzonego pliku:
```markdown
---
source: [nazwa pliku oryginalnego]
converted: clean | lossy
used-in: [lista faz]
---
```

Jeśli konwersja oznaczona jako `lossy` — zapisz do `state/inputs/processed/index.md`
flagę `requires-verification: true`. Orkiestrator zapyta użytkownika o weryfikację.

---

## Krok 4 — Zasilanie wsteczne

Jeśli użytkownik dostarczył materiały z wyższej fazy a niższa nie jest pokryta:

```
Design Level  →  ekstrahuj dane przydatne dla Process Level i Big Picture
Process Level →  ekstrahuj dane przydatne dla Big Picture
```

Zapisz wyekstrahowane dane do:
```
state/inputs/processed/phase-[N]-derived-for-[M].md
```

Nagłówek:
```markdown
---
derived-from: phase-[N]
target-phase: phase-[M]
---
```

---

## Krok 5 — Ocena pokrycia per faza

Na podstawie złożoności (Krok 1) i zawartości przetworzonych materiałów oceń pokrycie:

```
Big Picture   [X%]
  - aktorzy zidentyfikowani?
  - zdarzenia domenowe — liczba względem złożoności
  - granice systemu określone?
  - hot spoty wskazane?

Process Level [X%]
  - przepływy procesów opisane?
  - reguły biznesowe?
  - wyjątki i ścieżki alternatywne?
  - bounded contexts zarysowane?

Design Level  [X%]
  - agregaty zidentyfikowane?
  - encje i value objects?
  - kontrakty między kontekstami?
```

Zaktualizuj `state/inputs/processed/index.md` — uzupełnij kolumny pokrycia per plik.

---

## Krok 6 — Budowanie seed files

Dla każdej wybranej fazy skompiluj seed file z przetworzonych materiałów:

```
state/inputs/processed/phase-1-seed.md   ← jeśli BP wybrane
state/inputs/processed/phase-2-seed.md   ← jeśli PL wybrane
state/inputs/processed/phase-3-seed.md   ← jeśli DL wybrane
```

Format nagłówka seed:
```markdown
---
seed-for: big-picture | process-level | design-level
session: [nazwa]
coverage: [X%]
sources:
  - state/inputs/processed/[plik]-processed.md
  - state/inputs/processed/phase-[N]-derived-for-[M].md
---

# Seed: [nazwa fazy]

[skompilowana treść z wszystkich relevantnych przetworzonych plików]
```

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "data-prep",
  "status": "completed | completed-with-warnings | error",
  "complexity": "niska | średnia | wysoka",
  "coverage": {
    "big-picture": "[X%]",
    "process-level": "[X%]",
    "design-level": "[X%]"
  },
  "seed_files": {
    "phase-1": "state/inputs/processed/phase-1-seed.md",
    "phase-2": "state/inputs/processed/phase-2-seed.md",
    "phase-3": "state/inputs/processed/phase-3-seed.md"
  },
  "requires_verification": ["lista plików z lossy conversion"],
  "open_questions": [],
  "suggested_next": "big-picture | process-level | design-level",
  "user_decision_required": true
}
```

`status: completed-with-warnings` gdy którykolwiek plik ma `converted: lossy`.
`suggested_next` = najniższa faza z pokryciem > 0%.
