---
name: deep-analysis-data-prep
description: >
  Przetwarza materiały wejściowe dostarczone przez użytkownika przed uruchomieniem faz analizy.
  Uruchamiany przez orkiestrator w dwóch etapach: 'prepare' (składowanie, konwersja, normalizacja,
  ocena złożoności, zasilanie wsteczne) oraz 'seed' (kompilacja seed files po ocenie pokrycia).
  Ocena pokrycia NIE należy do tego skilla — wykonuje ją deep-analysis-coverage-assessment
  pomiędzy etapami. Zwraca handoff z wynikami etapu.
tools: Read, Write, Bash, Glob, LS
---

# Deep Analysis — Data Prep (Skill DP)

## Zasada nadrzędna

Ten skill przetwarza dane — nie analizuje domeny i **nie ocenia pokrycia**
(to odpowiedzialność `deep-analysis-coverage-assessment`).
Każdy krok tworzy pliki na dysku przed przejściem do następnego.
Awaria w połowie = odtworzenie od ostatniego zapisanego kroku.

Skill działa w dwóch etapach wywoływanych osobno przez orkiestrator:

```
etap: prepare  →  Kroki 0–4 (katalogi, złożoność, składowanie, konwersja, zasilanie wsteczne)
       ↓
[coverage-assessment — uruchamia orkiestrator]
       ↓
etap: seed     →  Krok 5 (kompilacja seed files z wartością coverage z index.md)
```

---

## Dane wejściowe od orkiestratora

```
Etap prepare:
- etap: prepare
- opis systemu (2-3 zdania z pytania orientacyjnego nr 3)
- ścieżka do material_input/ (materiały dostarczone przez użytkownika)
- ścieżka do katalogu state/
- lista faz wybranych przez użytkownika (BP / PL / DL)

Etap seed:
- etap: seed
- lista faz do zasilenia
- ścieżka do katalogu state/
```

---

# ETAP PREPARE

## Krok 0 — Utwórz strukturę katalogów

Przed jakimkolwiek przetwarzaniem utwórz katalogi dla tego kroku:

```
state/inputs/
  raw/
  processed/
```

Zasada ogólna: każdy skill tworzy własne katalogi na początku działania.
Nie zakładaj że katalog istnieje.

**Brak materiałów:** jeśli `material_input/` nie istnieje lub jest pusty — wykonaj tylko
Krok 1 (złożoność), zwróć handoff ze statusem `completed` i pustym manifestem.
Orkiestrator zdecyduje o starcie bez seedów.

---

## Krok 1 — Ocena złożoności systemu

Na podstawie opisu systemu (opis z pytania 3) oceń złożoność:

```
Złożoność niska:   jeden kontekst, mało integracji, prosta domena
Złożoność średnia: kilka kontekstów lub integracji, reguły biznesowe
Złożoność wysoka:  wiele kontekstów, złożone reguły, dużo integracji
```

To **jedyne** miejsce w systemie gdzie złożoność jest oceniana — orkiestrator
i pozostałe skille konsumują wynik, nie oceniają ponownie.

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

## Krok 2 — Składowanie oryginałów i manifest

Skopiuj wszystkie materiały z `material_input/` bez modyfikacji do `state/inputs/raw/`.
Zanotuj każdy plik w `state/inputs/processed/manifest.md` (tworzysz plik jeśli nie istnieje):

```markdown
---
generated-by: data-prep
---
# Inputs Manifest
| Plik | Typ | Konwersja | Użyty w fazach |
|---|---|---|---|
| [nazwa] | [typ] | clean / lossy | phase-1, phase-2 |
```

Manifest jest własnością data-prep. Ocena pokrycia trafia do osobnego pliku
`index.md`, którego właścicielem jest `deep-analysis-coverage-assessment`.

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

Jeśli konwersja oznaczona jako `lossy` — zapisz flagę `requires-verification: true`
w `manifest.md` i zwróć plik w polu `requires_verification` handoffu.
Orkiestrator zapyta użytkownika o weryfikację.

**Tagowanie typów zdarzeń:** przy ekstrakcji oznacz każde zdarzenie jako
`domenowe / zewnętrzne / czasowe`. Fazy w trybie audit konsumują tę klasyfikację
i nie wykonują własnej — zdarzenia zewnętrzne zasilają wprost analizę granic
(Krok 2a Big Picture).

**Cross-check ekstrakcji z opisem systemu:** porównaj wynik ekstrakcji (zwłaszcza
z obrazów) z opisem systemu. Obszar obecny w opisie, a nieobecny w ekstrakcji →
flaga `requires-verification` z adnotacją "prawdopodobna strata konwersji, nie luka
domenowa". Bez tego cross-checku audyt zgłosi fałszywe luki tam, gdzie zawiodła
konwersja, a nie wiedza interesariuszy.

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

Zasilanie wsteczne wykonuje się **przed** oceną pokrycia — coverage-assessment
widzi już pliki derived i ocenia pokrycie z ich uwzględnieniem (jeden przebieg oceny).

Koniec etapu prepare — zwróć handoff.

---

# ETAP SEED

## Krok 5 — Budowanie seed files

Wymaga istnienia `state/inputs/processed/index.md` (wynik coverage-assessment).
Jeśli nie istnieje — zwróć handoff ze statusem `error` i opisem braku.

Dla każdej wskazanej fazy skompiluj seed file z przetworzonych materiałów:

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
coverage: [X%]            ← wartość odczytana z index.md dla tej fazy
sources:
  - state/inputs/processed/[plik]-processed.md
  - state/inputs/processed/phase-[N]-derived-for-[M].md
---

# Seed: [nazwa fazy]

## Kontekst domeny
[dosłowny opis systemu z complexity.md — obowiązkowa pierwsza sekcja każdego seeda;
 obrazy i wyekstrahowane zdarzenia bez kontekstu tekstowego nie kalibrują analizy]

[skompilowana treść z wszystkich relevantnych przetworzonych plików]
```

---

## Zwróć handoff do orkiestratora

```json
{
  "skill": "data-prep",
  "etap": "prepare | seed",
  "status": "completed | completed-with-warnings | error",
  "complexity": "niska | średnia | wysoka",
  "seed_files": {
    "phase-1": "state/inputs/processed/phase-1-seed.md",
    "phase-2": "state/inputs/processed/phase-2-seed.md",
    "phase-3": "state/inputs/processed/phase-3-seed.md"
  },
  "requires_verification": ["lista plików z lossy conversion"],
  "open_questions": [],
  "user_decision_required": false
}
```

Zasady wypełniania:
- `complexity` — tylko w etapie prepare
- `seed_files` — tylko w etapie seed (w prepare: pusty obiekt)
- `requires_verification` — tylko w etapie prepare
- `status: completed-with-warnings` gdy którykolwiek plik ma `converted: lossy`
- Ocena pokrycia i `recommended_start_phase` NIE są zwracane przez ten skill —
  pochodzą z handoffu `deep-analysis-coverage-assessment`
