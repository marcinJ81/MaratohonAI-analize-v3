---
name: deep-analysis-big-picture
description: >
  Prowadzi fazę Big Picture Event Stormingu. Uruchamiaj jako Skill 1 gdy orkiestrator
  zleca analizę Big Picture. W trybie audit (główny): weryfikuje kompletność wsadu —
  timeline, granice, pivotal events, weryfikacja od tyłu — i generuje pytania o luki;
  wymaga seeda, bez niego zwraca needs-input. W trybie elicit: zbiera zdarzenia, aktorów,
  granice i hot spoty od zera. Obsługuje Tryby 1-4. Zapisuje wynik do state/phase-1-output.md.
tools: Read, Write, Bash, LS
---

# Deep Analysis — Big Picture (Skill 1)

## Zasada nadrzędna

Big Picture = szeroki obraz domeny bez zagłębiania się w szczegóły implementacyjne.
Celem jest zebranie zdarzeń domenowych, oś czasu, aktorów i granic — nie ich analiza.
Jeśli rozmowa schodzi w szczegóły procesu → zanotuj jako hot spot, wróć do szerokiego obrazu.

---

## Dane wejściowe od orkiestratora

```
- tryb pracy (1 / 2 / 3 / 4)
- tryb analizy (audit / elicit) + input-quality (z bramy B2 orkiestratora)
- cel analizy
- decyzja bleed-through (read modele / zasady biznesowe: tak / nie)
- state/inputs/processed/phase-1-seed.md (w trybie audit — WYMAGANY)
- ścieżka do katalogu state/
```

---

## Krok 0 — Weryfikacja wejścia i struktura katalogów

### 0a — Weryfikacja danych od orkiestratora

Sprawdź czy orkiestrator przekazał komplet danych wejściowych:

- **tryb pracy (1/2/3/4)** — jeśli brak → zapytaj użytkownika który tryb wybiera
- **cel analizy** — jeśli brak → zapytaj: "Co jest celem tej analizy — zrozumienie domeny,
  decyzja architektoniczna, prezentacja dla biznesu, coś innego?"
- **decyzja bleed-through** — jeśli brak → zaproponuj użytkownikowi:

```
Big Picture bazowe (domyślne):
  zdarzenia domenowe, aktorzy, systemy zewnętrzne, hot spoty, granice domeny

Opcjonalnie można dodać (zapytaj czy chcesz):
  [+] Read modele — co jest wyświetlane / prezentowane użytkownikowi w kluczowych momentach
  [+] Ogólne zasady biznesowe — reguły które wpływają na przepływ, bez szczegółów implementacji
```

  Zapisz decyzję — będzie potrzebna w Krokach 4 i 7 oraz w handoffie.
- **ścieżka do state/** — jeśli brak → użyj bieżącego katalogu roboczego, odnotuj to w handoffie

Nie ruszaj dalej dopóki tryb i cel nie są znane.

### 0b — Struktura katalogów

```
state/
  phase-1/
    working/     ← pliki robocze zbierane w trakcie sesji
```

---

## Krok 1 — Załaduj kontekst i chaotyczna eksploracja

**Reguła trybu analizy:**

```
tryb audit + brak phase-1-seed.md → zwróć handoff:
  { "phase": "big-picture", "status": "needs-input",
    "open_questions": ["Tryb audit wymaga materiałów wejściowych —
     dostarcz wsad albo przełącz na tryb elicit (jawna decyzja)."] }
  i ZAKOŃCZ. Żadnego cichego przejścia do elicytacji.

tryb audit + seed istnieje → wyłącznie gałąź "praca na materiale" (poniżej)
tryb elicit               → obie gałęzie wg dostępności seeda
```

### Tylko tryb elicit — chaotyczna eksploracja (seed nie istnieje)

Nim rozpoczniesz generowanie zdarzeń zadaj kilka pytań:
- o kontekst systemu, do czego służy, jak wygląda proces; nie oczekuj opisu całego systemu,
  ale na podstawie odpowiedzi możesz zawęzić wygenerowane zdarzenia
- jeżeli użytkownik poda bardzo krótki opis, albo nie poda go wcale, spróbuj generować zdarzenia
  na podstawie tego co masz, bo zapewne dostaniesz coś na wstępie z orkiestratora
- ten etap nie musi być dokładny, ważne są zdarzenia; jeżeli nie są trafione,
  na późniejszych etapach będą usunięte

Nie narzucaj struktury. Zbieraj zdarzenia swobodnie, bez porządku chronologicznego.

**Klasyfikacja typów zdarzeń:** przy każdym zebranym zdarzeniu oznacz czy to:
- zdarzenie domenowe (domain event)
- zdarzenie zewnętrzne (z systemu zewnętrznego)
- zdarzenie czasowe (time-based / scheduled)

Jeżeli masz dane wsadowe i tryb agentowy — dostosuj typ zdarzeń do tego co masz na podstawie wsadu.

Klasyfikacja to oznaczenie, nie analiza — nie zatrzymuj eksploracji na dyskusję o typach.
Zdarzenia oznaczone jako zewnętrzne będą wejściem do Kroku 2a (granice zewnętrzne).

```
"Wyrzuć wszystko co wiesz o tym systemie — zdarzenia, fakty, problemy, przepływy.
 Nie musisz zachować kolejności. Zapisuj w czasie przeszłym.
 Zatrzymamy się gdy skończy się materiał."
```

Kontynuuj dopóki użytkownik nie powie "gotowe" lub nie doda nowych zdarzeń przez dwie rundy.
Zapisz surowy wynik do `state/phase-1/working/raw-events.md`.

### Praca na materiale wejściowym (tryb audit; elicit z seedem)

Wczytaj `state/inputs/processed/phase-1-seed.md`. Sekcja "Kontekst domeny" z seeda
jest podstawą kalibracji — przeczytaj ją przed zdarzeniami.
Wyświetl użytkownikowi podsumowanie: ile zdarzeń / aktorów / hot spotów już jest.
Zapytaj: "Chcesz uzupełnić te dane czy coś korygować?"

Nie generuj własnych zdarzeń poza materiałem — seed jest źródłem prawdy dla tego kroku.
Typy zdarzeń (domenowe / zewnętrzne / czasowe) powinny być już otagowane przez data-prep;
jeśli nie są — sklasyfikuj przy wczytaniu (to oznaczenie, nie analiza).
Zapisz do `state/phase-1/working/raw-events.md`.

W trybie audit po tym kroku przejdź wprost do Kroku 2 — punkt ciężkości fazy
przesuwa się na kroki weryfikacyjne (2a, 2b, 4, 5), które działają bez zmian.

---

## Krok 2 — Timeline — oś czasu

Na podstawie zebranych zdarzeń (Krok 1) wprowadź oś czasu.

```
"Ułóżmy zdarzenia w kolejności — co się dzieje najpierw, co potem?
 Jeśli coś dzieje się równolegle — oznacz jako równoległe.
 Nie musimy znać dokładnych dat — wystarczy kolejność."
```

Pogrupuj zdarzenia wzdłuż osi czasu. Oznacz:
- zdarzenia sekwencyjne
- zdarzenia równoległe
- zdarzenia cykliczne (np. harmonogram)

Na tym etapie można zweryfikować czy wszystkie zdarzenia są w kontekście całej aplikacji, systemu.
Wygeneruj użytkownikowi zdarzenia, zapytaj czy są takie które nie pasują do kontekstu.

Zapisz do `state/phase-1/working/timeline.md`.

---

## Krok 2a — Granice i pivotal events

Na podstawie uporządkowanej osi czasu:

### 1. Pivotal events

Odkryj pivotal events — zdarzenia po których stan systemu zmienia się fundamentalnie.
To naturalni kandydaci na granice kontekstów. Oznacz je wyraźnie na osi czasu.

### 2. Grupy zdarzeń — granice wewnętrzne

W tym momencie widać że zdarzenia przybierają postać grup — jedne pasują do siebie, inne nie.
Takie grupy mogą wynikać z różnych kontekstów — zaznacz je.
Sprawdź czy pivotal events pokrywają się z granicami grup — jeśli tak, to silny sygnał granicy kontekstu.

### 3. Granice zewnętrzne — systemy zewnętrzne

Wykorzystaj zdarzenia oznaczone w Kroku 1 jako zewnętrzne. Dla każdego określ:
- jaki system zewnętrzny
- w którym miejscu osi czasu wchodzi w interakcję
- kierunek interakcji (wchodzi / wychodzi / dwukierunkowo)

Zapytaj:
```
"Co jest w systemie a co poza nim?
 Czy są systemy zewnętrzne których nie widać w zebranych zdarzeniach?"
```

### Wyniki

Zaktualizuj `state/phase-1/working/timeline.md` (pivotal events, grupy).
Zapisz granice zewnętrzne do `state/phase-1/working/boundaries.md`.

---

## Krok 2b — Język wszechobecny (Ubiquitous Language) — weryfikacja

Przegląd nazw zebranych zdarzeń pod kątem języka: czy używają języka biznesowego czy technicznego.
Jeśli zdarzenie brzmi jak `UpdateDatabaseRecord` zamiast `ZamówienieZatwierdzone` — to sygnał do korekty.

Zapytaj przy wątpliwych nazwach:
```
"Czy te nazwy są zrozumiałe dla osoby biznesowej bez wiedzy technicznej?"
```

### Wyniki

Zaktualizuj `state/phase-1/working/timeline.md`.

---

## Krok 3 — Hot spoty i aktorzy

### 3a — Hot spoty

Podczas Kroków 1-2 oznaczaj hot spoty na bieżąco.
Hot spot = miejsce gdzie:
- użytkownik się waha lub mówi "to zależy"
- pojawia się konflikt między obszarami
- brakuje wiedzy domenowej
- zdarzenie wywołuje wiele pytań
- nie naciskaj — jeżeli użytkownik nie jest pewny, zasugeruj hot spota
- na każdym kolejnym etapie Big Picture można jeszcze dorzucić hot spota,
  albo oznaczyć jako `resolved` jeżeli się wyjaśni (nie usuwaj — zachowaj track decyzji)

Na końcu tego kroku zapytaj:
```
"Czy są miejsca w domenie które budzą wątpliwości,
 są niejasne lub wymagają decyzji biznesowej?"
```

Zapisz do `state/phase-1/working/hotspots.md`.

### 3b — Aktorzy

```
"Kto lub co wywołuje te zdarzenia?
 Może być człowiek, system zewnętrzny, harmonogram (czas), inny system."
```

Przypisz każdego aktora do zdarzeń z osi czasu. Oznacz typ aktora: człowiek / system / czas.
Mogą być zdarzenia inicjalizowane przez wielu aktorów — dowiedz się czy takie istnieją.
Jeżeli robisz to z gotowych materiałów i widzisz że są zdarzenia generowane przez wielu aktorów:
- w trybie agentowym: spróbuj na podstawie zdarzenia i kontekstu znaleźć czy jest więcej
  inicjatorów danego zdarzenia; nie rób tego na siłę do każdego zdarzenia — jeżeli widzisz
  takie zachowanie, spróbuj znaleźć dodatkowe możliwości
- w innym trybie: zapytaj użytkownika czy możliwe jest więcej inicjatorów danego zdarzenia

Zapisz do `state/phase-1/working/actors.md`.

---

## Krok 4 — Przejście przez proces do końca

Przejdź przez oś czasu od początku do końca zdarzenie po zdarzeniu.

Dla każdego zdarzenia zweryfikuj:
- czy ma aktora
- czy jest połączone z następnym zdarzeniem (co je wyzwala)
- czy nie brakuje zdarzeń pośrednich
- czy to jedyna ścieżka
- czy zawsze jest dany rezultat/zdarzenie, czy mogą być inicjalizowane

```
"Przejdziemy przez proces od początku do końca.
 Dla każdego zdarzenia — czy to wszystko co się dzieje w tym miejscu?
 Czy coś jest pominięte zanim przejdziemy dalej?"
```

**Jeśli wybrano bleed-through read modeli (Krok 0):** przy kluczowych zdarzeniach
(szczególnie pivotal events) zadaj dodatkowo:
```
"Czy w tym miejscu ktoś musi coś zobaczyć żeby podjąć decyzję?
 Co musi być widoczne i dla kogo?"
```
Zapisuj do `state/phase-1/working/read-models.md`.

**Jeśli wybrano bleed-through zasad biznesowych:** gdy użytkownik wspomina regułę
wpływającą na przepływ, zapisuj jako "reguła: [treść]" — bez szczegółów implementacji.
Zapisuj do `state/phase-1/working/business-rules.md`.

Uzupełnij brakujące zdarzenia, aktorów, hot spoty zgodnie z tabelą aktualizacji (poniżej Kroku 5).

---

## Krok 5 — Weryfikacja od tyłu

**Ten krok jest obowiązkowy gdy istnieje seed (materiały wsadowe).**
W pozostałych trybach — wykonaj gdy oś czasu ma więcej niż 10 zdarzeń lub użytkownik o to poprosi.

Przejdź przez oś czasu od końca do początku.
Jeżeli pracujesz w trybie agentowym, to jest to najbardziej istotny etap — na podstawie procesu
który odkryłeś (sam lub na podstawie danych) postaraj się zweryfikować zdarzenia.

```
"Wróćmy przez proces od tyłu.
 Dla każdego zdarzenia — co musiało zajść wcześniej żeby to zdarzenie mogło wystąpić?
 Czy wszystkie warunki wstępne są na osi czasu?"
```

Oznacz miejsca gdzie:
- brakuje zdarzenia wyzwalającego
- warunek wstępny nie wynika z wcześniejszych zdarzeń
- kolejność okazuje się błędna
- zweryfikuj proces pod kątem istnienia dodatkowych edge case'ów — rzadkich ścieżek

Istotne w trybie agentowym bez wsadu:
- przejdź happy path
- a potem zbadaj inne ścieżki które powinny wyjść przy cofaniu się od końca do początku

Jeżeli mamy wsad:
- zbadaj czy masz do czynienia z całym procesem czy jest tylko happy path
- szukaj trudnych ścieżek procesu, które nie zostały przedstawione w materiałach wsadowych
- nie opuszczaj kontekstu, nie zmyślaj, oceń prawdopodobieństwo wystąpienia;
  ten punkt jest istotny dla trybu agentowego bo ty musisz to zrobić na podstawie
  zebranych danych — w innych trybach to użytkownik określa

Dodatkowe ścieżki procesu:
- na koniec w wynikach oznacz proces: gdzie się zaczyna, co go wyzwala
  (zdarzenie, aktor, system zewnętrzny)
- jeżeli nie jesteś pewny rezultatu albo samego procesu — nie zgaduj,
  daj hot spota do wyjaśnienia potem

Zaktualizuj pliki zgodnie z tabelą aktualizacji (poniżej).

---

## Aktualizacja plików po zmianach (obowiązuje w Krokach 4 i 5)

Każde odkrycie ma przypisaną akcję — wykonuj deterministycznie, nie uznaniowo:

| Typ odkrycia | Akcja |
|---|---|
| nowe zdarzenie | dopisz do `raw-events.md` + `timeline.md` (z typem: domenowe/zewnętrzne/czasowe) |
| nowy aktor | dopisz do `actors.md` + przypisz do zdarzeń na osi czasu |
| nowy system zewnętrzny | dopisz do `boundaries.md` (system, miejsce na osi, kierunek) |
| hot spot rozwiązany | oznacz w `hotspots.md` jako `[resolved: powód]` — nie usuwaj |
| nowy hot spot | dopisz do `hotspots.md` |
| zmiana kolejności zdarzeń | zaktualizuj `timeline.md`, sprawdź czy pivotal events nadal pasują |
| nowy pivotal event | zaktualizuj `timeline.md`, zweryfikuj grupy kontekstów z Kroku 2a |
| nowy read model (jeśli bleed-through) | dopisz do `read-models.md` |
| nowa reguła biznesowa (jeśli bleed-through) | dopisz do `business-rules.md` |

---

## Krok 6 — Synteza i zapis wyniku

Wczytaj szablon: `templates/phase-1-template.md`.
Wypełnij szablon danymi z plików working/:

```
state/phase-1/working/raw-events.md      → surowy materiał (zachowaj do tracku)
state/phase-1/working/timeline.md        → sekcja Zdarzenia domenowe (chronologicznie,
                                           z pivotal events i grupami kontekstów)
state/phase-1/working/actors.md          → sekcja Aktorzy
state/phase-1/working/boundaries.md      → sekcja Granice systemu
state/phase-1/working/hotspots.md        → sekcja Hot Spoty (w tym resolved)
state/phase-1/working/read-models.md     → sekcja Read Modele (jeśli bleed-through)
state/phase-1/working/business-rules.md  → sekcja Zasady biznesowe (jeśli bleed-through)
```

Wygeneruj ID dla każdego elementu (A-001, E-001, H-001, PE-001 dla pivotal events).
Prefiks `PE-` jest globalny — Faza 2 kontynuuje numerację zamiast nadawać własne ID.
Wypełnij frontmatter: `input-quality` i `lossy-files` wartościami z bramy B2 orkiestratora
(`n/a` w trybie elicit).
Wypełnij sekcję Wnioski — minimum 3 wnioski z oceną pewności.
**Reguła pewności:** wniosek oparty wyłącznie na materiale `lossy` → pewność maksymalnie `średnia`.
Wypełnij sekcję Otwarte pytania — oznacz blokery.

Zapisz do `state/phase-1-output.md`.

---

## Krok 7 — Weryfikacja z użytkownikiem

Wyświetl podsumowanie:
```
Zebrano:
- Aktorów: [N]
- Zdarzeń domenowych: [N]
- Pivotal Events: [N]
- Granic / integracji: [N]
- Hot Spotów: [N] (w tym resolved: [N])
- Blokerów: [N]
- Read modeli: [N] (jeśli bleed-through)
- Zasad biznesowych: [N] (jeśli bleed-through)
```

Zapytaj: "Czy wynik odzwierciedla Twoją wiedzę o domenie? Co wymaga korekty?"
Nanieś korekty zgodnie z tabelą aktualizacji i zapisz zaktualizowany `state/phase-1-output.md`.

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "big-picture",
  "status": "completed",
  "output_file": "state/phase-1-output.md",
  "pivotal_events": ["PE-001: nazwa zdarzenia", "PE-002: nazwa zdarzenia"],
  "actors": [
    {"id": "A-001", "name": "nazwa", "type": "człowiek|system|czas"}
  ],
  "hotspots": [
    {"id": "H-001", "desc": "opis", "status": "open|resolved", "priority": "blocker|wysoki|średni|niski"}
  ],
  "external_systems": ["nazwa systemu (kierunek interakcji)"],
  "bleed_through": {"read_models": false, "business_rules": false},
  "open_questions": ["lista pytań z phase-1-output.md gdzie priorytet = blocker"],
  "suggested_next": "process-level",
  "user_decision_required": true
}
```
