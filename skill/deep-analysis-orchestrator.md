---
name: deep-analysis-orchestrator
description: >
  Orkiestrator procesu głębokiej analizy domenowej. Uruchamiaj jako pierwszy gdy użytkownik chce
  dogłębnie zrozumieć problem, domenę lub architekturę: "przeanalizujmy to od podstaw", "chcę
  zrozumieć domenę", "zróbmy analizę", "jak to powinno działać", "zacznijmy od big picture",
  "zamodelujmy to", "rozbijmy to na części", "mam system i chcę zrozumieć jak działa".
  Użyj też gdy sub-skill zakończył fazę i zwrócił handoff JSON — odbierz wynik i zdecyduj co dalej.
tools: Read, Write, Edit, Bash, Glob, LS, Task
---

# Deep Analysis — Orkiestrator (Skill 0)

## Zasada nadrzędna

Analiza to pętla zwrotna, nie liniowy proces.
Orkiestrator nie analizuje domeny — zarządza procesem, stanem i decyzjami o przejściu między fazami.
Sub-skille analizują. Orkiestrator koordynuje.
Orkiestartor zapisuje w projekcie plik loga, będzie tam zapisywana każda akcja która została wykonana przez orkiestrator w formacie data, godzina, krótko co zostało zrobione (dwa, trzy słowa)

---

## Uruchomienie — pierwsze wywołanie

### Krok 1: Sprawdź czy sesja istnieje

Odczytaj `state/session.md`:
- Jeśli plik nie istnieje → nowa sesja, przejdź do **Inicjalizacji**
- Jeśli plik istnieje → wznowienie sesji, przejdź do **Wznowienia**

---

### Krok 1a: przeszukaj folder material_input

Powinien być w projekcie w którym aktualnie będziesz prowadził analizę.
Jeżeli go nie ma lub jest pusty, wyświetl użytkownikowi informację że nie znalazłeś materiałów wejściowych do analizy.

### Krok 2a: Inicjalizacja nowej sesji

#### Pytania orientacyjne

Zadaj tylko te, których nie można wywnioskować z kontekstu, zadaj jedno pytanie oczekuj odpowiedzi, potem kolejne aż do piątego:

```
1. Co jest przedmiotem analizy — system, proces, decyzja architektoniczna?
2. Jaki jest cel — zrozumienie domeny, planowanie, decyzja, coś innego?
3. Opisz system w 2-3 zdaniach: co robi i z czym się łączy?
4. Czy istnieje już Event Storming lub inna analiza wejściowa?
5. Jaki poziom szczegółowości — tylko Big Picture, do Process Level, pełna analiza?
```
Po uzyskaniu odpowiedzi na wszystkie pytania rozpocznij wstępną ocenę złożoności systemu:

```
Złożoność niska:   jeden kontekst, mało integracji, prosta domena
Złożoność średnia: kilka kontekstów lub integracji, reguły biznesowe
Złożoność wysoka:  wiele kontekstów, złożone reguły, dużo integracji
```

Wynik złożoności przekaż jako parametr do `deep-analysis-coverage-assessment`.

---

#### Brak materiałów wejściowych

Zasugeruj start od Big Picture w Trybie 2.
Ale pozwól wybrać użytkownikowi Tryb pracy — patrz sekcja Tryb Pracy, nie wybieraj za niego nawet
jeżeli brak jest danych albo są niekompletne albo kompletne.

Jeśli użytkownik wybierze inny tryb lub inną fazę startową — poinformuj że wyniki mogą być
niewystarczające. Kontynuuj zgodnie z decyzją użytkownika.

Agent fazy zbiera odpowiedzi użytkownika i zapisuje je na bieżąco jako pliki MD z nagłówkiem:

```markdown
<!-- source: user-input, session: [nazwa], phase: [faza] -->
<!-- used-in: [lista faz które otrzymają ten plik jako seed] -->
```

Orkiestrator pilnuje że na wejściu do każdej fazy i na wyjściu z sesji istnieją pliki
z zebranymi danymi — chyba że użytkownik jawnie zdecyduje inaczej.

---

#### Materiały wejściowe dostarczone przez użytkownika

##### Krok A — Składowanie oryginałów

Zapisz wszystkie materiały bez modyfikacji:

```
state/inputs/raw/
  [nazwa-pliku-oryginał].[ext]
```

##### Krok B — Konwersja i normalizacja

Dla każdego materiału:

| Typ | Akcja |
|---|---|
| Obraz / screenshot | Opisz zawartość, wyekstrahuj zdarzenia, aktorów, granice. Oznacz jako `converted: lossy` jeśli struktura mogła zostać utracona. |
| Tekst niestrukturalny | Znormalizuj do formatu MD z sekcjami: Zdarzenia, Aktorzy, Granice, Hot Spoty, Pytania otwarte. |
| Gotowy ES / diagram | Wyekstrahuj dane per kategoria jak wyżej. |

Zapisz wynik do:

```
state/inputs/processed/
  [nazwa-pliku]-processed.md
```

Jeśli konwersja była oznaczona jako `lossy` — zapytaj użytkownika o weryfikację opisu
przed dalszym użyciem.

##### Krok C — Ocena pokrycia (delegowana do sub-skilla)

Po zakończeniu Kroku B uruchom sub-skill `deep-analysis-coverage-assessment` jako Task:

```
Wejście:
  - state/inputs/processed/    ← wszystkie pliki po normalizacji
  - złożoność systemu          ← niska / średnia / wysoka (z pytania 3)

Oczekiwane wyjście:
  - state/inputs/processed/index.md   ← tabela pokrycia + szczegóły per faza
  - handoff JSON                      ← coverage%, recommended_start_phase, blockers
```

Po zakończeniu sub-skilla:
1. Wyświetl użytkownikowi tabelę pokrycia z `index.md`
2. Uwzględnij `blockers` z handoff JSON przy pytaniu o tryb pracy
3. Przekaż `recommended_start_phase` jako sugestię (nie decyzję — użytkownik decyduje)

##### Krok D — Zasilanie wsteczne

Jeśli użytkownik dostarczył materiały z wyższej fazy a niższa nie istnieje:

```
Design Level  →  ekstrahuj dane przydatne dla Process Level i Big Picture
Process Level →  ekstrahuj dane przydatne dla Big Picture
```

Dane oznacz jako `derived-from: phase-[N]` i zapisz do:

```
state/inputs/processed/
  phase-[N]-derived-for-[M].md
```

Po uzupełnieniu danymi z zasilania wstecznego — uruchom `deep-analysis-coverage-assessment`
ponownie, przekazując zaktualizowany katalog `state/inputs/processed/`.
Zaktualizowany `index.md` zastępuje poprzedni wynik.

##### Krok E — Przygotowanie seed dla agenta (data-prep)

Przed uruchomieniem każdej fazy orkiestrator wykonuje krok data-prep:

```
Wejście:  state/inputs/processed/ — wszystkie pliki dotyczące danej fazy
Wyjście:  state/inputs/processed/phase-[X]-seed.md
Zawiera:  skompilowane dane wejściowe + ocena pokrycia + oznaczenia źródeł
```

Format nagłówka seed:

```markdown
# Seed: Phase [X] — [nazwa fazy]
<!-- generated-by: orchestrator, session: [nazwa] -->
<!-- coverage: [X%] -->          ← wartość z index.md dla tej fazy
<!-- sources: [lista plików źródłowych] -->
```

Agent fazy startuje z tym plikiem jako pierwszym kontekstem.

---

#### Struktura katalogów po Kroku 2a

```
state/
  inputs/
    raw/                           ← oryginały użytkownika, bez modyfikacji
    processed/
      index.md                     ← lista materiałów + ocena % per faza (z coverage-assessment)
      [nazwa]-processed.md         ← znormalizowane materiały
      phase-[N]-derived-for-[M].md ← dane z zasilania wstecznego
      phase-1-seed.md              ← seed dla Big Picture
      phase-2-seed.md              ← seed dla Process Level (jeśli dane istnieją)
  session.md
```

`index.md` jest generowany wyłącznie przez `deep-analysis-coverage-assessment`.
Orkiestrator nie zapisuje do tego pliku bezpośrednio.

---

### Krok 2b: Wznowienie sesji

Odczytaj `state/session.md`. Podsumuj użytkownikowi aktualny stan:
- które fazy są zakończone
- która faza była aktywna
- otwarte pytania przeniesione z ostatniej fazy

Zapytaj: "Czy wznawiamy od miejsca gdzie skończyliśmy, czy chcesz zmienić kierunek?"

---

## Tryby pracy

Po orientacji, przed uruchomieniem pierwszego sub-skilla, zaproponuj tryb.
Jeśli użytkownik nie wybiera → zastosuj **Tryb 2** jako domyślny i poinformuj o tym.
Tryb można zmienić w dowolnym momencie ("przełącz na tryb X").

### Tryb 1 — Agentowy (Claude orkiestruje)
Claude dzieli domenę na konteksty i przydziela je sub-agentom.
Kiedy warto: duża domena, wiele kontekstów, użytkownik chce szybkiego pokrycia.

### Tryb 2 — Ekspercki (użytkownik jako ekspert domenowy) ← domyślny
Sub-agent zadaje pytania, użytkownik odpowiada.
Claude śledzi spójność między odpowiedziami i sygnalizuje konflikty.
Kiedy warto: użytkownik ma głęboką wiedzę domenową i chce ją wyekstrahować krok po kroku.

### Tryb 3 — Mixed (współpraca)
Część kontekstów trafia do sub-agentów, część użytkownik prowadzi sam.
Kiedy warto: niejednorodna domena, część obszarów wrażliwa lub wymaga unikalnej wiedzy.

### Tryb 4 — Użytkownik jako orkiestrator
Użytkownik decyduje co i kiedy badać, sub-agenci dostają konteksty od użytkownika.
Claude reaguje na pytania i pilnuje spójności na żądanie.
Kiedy warto: użytkownik ma gotowy plan i chce narzędzia, nie prowadzenia.

---

## Pliki skilli w repo

```
deep-analysis-big-picture          ← Big Picture (główny plik)
  phase-1-template                 ← szablon Big Picture
deep-analysis-specialist           ← Process Level
  phase-2-template                 ← szablon Process Level
deep-analysis-design-level         ← Design Level
  phase-3-template                 ← szablon Design Level

deep-analysis-coverage-assessment  ← ocena pokrycia faz materiałami ← NOWY
deep-analysis-data-prep            ← przygotowanie danych
deep-analysis-specialist
deep-analysis-llm-blueprint
deep-analysis-mermaid-generator
deep-analysis-orchestrator
```

---

## Mapa zdarzeń → akcje

Każdy sub-skill po zakończeniu zwraca handoff JSON. Orkiestrator interpretuje go według tej mapy.

### Schemat handoff JSON — fazy analityczne

```json
{
  "phase": "big-picture | process-level | design-level | specialist",
  "status": "completed | paused | needs-input | error",
  "output_file": "state/phase-[X]-output.md",
  "open_questions": ["lista otwartych pytań"],
  "suggested_next": "process-level | design-level | specialist | end",
  "user_decision_required": true
}
```

### Schemat handoff JSON — coverage-assessment

```json
{
  "skill": "coverage-assessment",
  "status": "completed",
  "output_file": "state/inputs/processed/index.md",
  "coverage": {
    "big_picture": 85,
    "process_level": 40,
    "design_level": 10
  },
  "recommended_start_phase": "big-picture",
  "blockers": ["lista brakujących danych które blokują start fazy"],
  "user_decision_required": false
}
```

### status == "completed" (fazy analityczne)

Jeśli `user_decision_required == true` → wyświetl `open_questions`, czekaj na potwierdzenie
użytkownika przed przejściem dalej.

```
suggested_next == "process-level"  → zaproponuj uruchomienie skill-2-process-level
suggested_next == "design-level"   → zaproponuj uruchomienie skill-3-design-level
suggested_next == "specialist"     → zapytaj którą fazę specjalistyczną (S1/S2/S3)
suggested_next == "end"            → podsumuj sesję, zaktualizuj session.md, zakończ
```

### status == "completed" (coverage-assessment)

Wyświetl tabelę pokrycia. Jeśli `user_decision_required == true` — wyświetl pytanie z handoff,
zbierz odpowiedź, przekaż do coverage-assessment jako kontynuację.
Następnie przejdź do Kroku D lub Kroku E zgodnie z pipeline.

### status == "paused"

Wyświetl `open_questions`. Czekaj na odpowiedź użytkownika.
Przekaż odpowiedź do tego samego sub-skilla jako kontynuację.

### status == "needs-input"

Wyświetl pytanie z handoff. Zbierz odpowiedź.
Uruchom ten sam sub-skill z odpowiedzią jako danymi wejściowymi.

### status == "error"

Wyświetl opis błędu. Zapytaj: "Powtórzyć fazę, pominąć, czy zakończyć sesję?"

---

## Reguły przejść między fazami

Przejście jest możliwe tylko gdy warunki są spełnione.
Jeśli nie są — poinformuj użytkownika i zapytaj czy kontynuować mimo to.

| Z fazy | Do fazy | Wymagane warunki |
|---|---|---|
| Big Picture | Process Level | wnioski zapisane w phase-1-output.md + brak krytycznych open_questions |
| Process Level | Design Level | zidentyfikowane BC + nazwana core domain w phase-2-output.md |
| Dowolna | Specialist (S1–S3) | na żądanie użytkownika, w każdym momencie |
| Dowolna | End | decyzja użytkownika |

---

## Uruchamianie sub-skillów

Uruchamiaj sub-skille jako Task z odpowiednimi danymi wejściowymi.

### Dane wejściowe dla każdego sub-skilla

```
Skill 0a (Coverage Assessment):
  - state/inputs/processed/       ← katalog z przetworzonymi materiałami
  - złożoność systemu             ← niska / średnia / wysoka

Skill 1 (Big Picture):
  - tryb pracy
  - cel analizy
  - state/inputs/processed/phase-1-seed.md (jeśli istnieje)
  - ścieżka do katalogu state/

Skill 2 (Process Level):
  - tryb pracy
  - state/inputs/processed/phase-2-seed.md (jeśli istnieje)
  - ścieżka do state/phase-1-output.md
  - otwarte pytania z Fazy 1

Skill 3 (Design Level):
  - tryb pracy
  - state/inputs/processed/phase-3-seed.md (jeśli istnieje)
  - ścieżka do state/phase-1-output.md
  - ścieżka do state/phase-2-output.md
  - lista bounded contexts do analizy

Skill 4 (Specialist):
  - tryb pracy
  - która faza specjalistyczna (S1/S2/S3)
  - ścieżki do wszystkich dostępnych phase-N-output.md
```

---

## Pętla zwrotna

Jeśli wynik z sub-skilla zmienia wnioski z poprzedniej fazy:
1. Otwórz plik MD tamtej fazy
2. Dodaj sekcję `## Korekty i uzupełnienia (retrospektywne)` z formatem:

```markdown
[kontekst]: Wniosek [ID] wymaga korekty po Fazie [N].
Poprzednio: [stary wniosek]
Aktualnie: [nowy wniosek]
Powód: [co zmieniło ocenę]
```

3. Zaktualizuj `state/session.md`

---

## Format state/session.md

Utwórz ten plik przy inicjalizacji. Aktualizuj po każdym zakończonym sub-skilla.

```markdown
# Session: [nazwa domeny / projektu]
Data startu: [data]
Tryb: [Tryb N]
Złożoność systemu: [niska / średnia / wysoka]

## Cel analizy
[jeden akapit]

## Fazy
| Faza | Status | Plik outputu | Seed |
|---|---|---|---|
| Big Picture | not-started / in-progress / completed | state/phase-1-output.md | state/inputs/processed/phase-1-seed.md |
| Process Level | not-started / in-progress / completed | state/phase-2-output.md | state/inputs/processed/phase-2-seed.md |
| Design Level | not-started / in-progress / completed | state/phase-3-output.md | state/inputs/processed/phase-3-seed.md |
| Specialist | not-started / in-progress / completed | state/phase-s-output.md | — |

## Materiały wejściowe
<!-- wypełniane przez coverage-assessment — nie edytuj ręcznie -->
<!-- źródło: state/inputs/processed/index.md -->
| Plik | Typ | Konwersja | Pokrycie BP | Pokrycie PL | Pokrycie DL |
|---|---|---|---|---|---|

## Otwarte pytania (przeniesione między fazami)
- [pytanie]

## Decyzje użytkownika
- Tryb pracy: [N]
- Core domain: [nazwa]
- [inne decyzje]

## Historia handoffów
- [data] coverage-assessment → completed, BP: [X%], PL: [X%], DL: [X%]
- [data] Faza 1 → completed, suggested_next: process-level
- [data] Faza 2 → paused, open_questions: [...]
```

---

## Zakończenie sesji

Gdy użytkownik mówi "stop", "zakończ", "koniec" lub handoff zwraca `suggested_next == "end"`:

1. Zaktualizuj `state/session.md` — wszystkie statusy
2. Wypisz podsumowanie: które fazy zakończono, kluczowe wnioski per faza, otwarte pytania
3. Wskaż pliki MD które powstały w sesji

---

## Format plików wyjściowych

Każdy plik output fazy używa Structured Markdown z YAML frontmatter.
Format jest jednolity dla wszystkich faz — LLM parsuje go przewidywalnie,
człowiek czyta jako dokumentację.

### Nagłówek YAML (obowiązkowy)

```markdown
---
phase: big-picture | process-level | design-level
session: [nazwa projektu]
generated: [data]
status: completed | partial
coverage: [X%]
sources:
  - state/inputs/processed/phase-X-seed.md
  - user-input
---
```

### Sekcje wspólne dla każdej fazy

```markdown
## Kontekst
[2-3 zdania: co analizowano, cel, złożoność]

## Wnioski
| ID    | Wniosek | Pewność               | Źródło                  |
|-------|---------|-----------------------|-------------------------|
| W-001 | ...     | wysoka / średnia / niska | user-input / derived |

## Otwarte pytania
| ID    | Pytanie | Priorytet              | Dotyczy   |
|-------|---------|------------------------|-----------|
| Q-001 | ...     | blocker / nice-to-have | [obszar]  |

## Korekty i uzupełnienia (retrospektywne)
<!-- wypełnia orkiestrator gdy późniejsza faza zmienia ten output -->
<!-- format: [kontekst]: Wniosek [ID] wymaga korekty po Fazie [N]. -->
<!-- Poprzednio: [stary wniosek] / Aktualnie: [nowy wniosek] / Powód: [...] -->
```

### Sekcje właściwe per faza

**Big Picture (`phase-1-output.md`)**
- Aktorzy `| ID | Nazwa | Rola | Zewnętrzny? |`
- Zdarzenia domenowe `| ID | Zdarzenie | Aktor | Obszar |`
- Granice systemu + lista integracji zewnętrznych
- Hot Spoty `| ID | Opis | Powód | Powiązane zdarzenia |`

**Process Level (`phase-2-output.md`)**
- Bounded Contexts `| ID | Nazwa | Odpowiedzialność | Core / Supporting / Generic |`
- Core Domain — nazwa + uzasadnienie
- Przepływy procesów — kroki numerowane + reguły biznesowe + wyjątki
- Zależności między kontekstami `| Z | Do | Typ relacji | Kierunek |`

> Szczegółowe szablony per faza: `templates/phase-1-template.md`, `templates/phase-2-template.md`

---

## Relacje z innymi skillami systemu

Przeszukaj lokalnego Claude'a w poszukiwaniu skilli które umożliwią implementację założeń.
Nie uruchamiaj — wyświetl je jako opcje do kontynuacji po zakończonej analizie.
