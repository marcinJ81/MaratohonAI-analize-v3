---
name: deep-analysis-process-level
description: >
  Prowadzi fazę Process Level Event Stormingu. Uruchamiaj jako Skill 2 gdy orkiestrator
  zleca analizę Process Level. Wymaga ukończonego Big Picture (phase-1-output.md).
  Odkrywa subdomeny i ich typy, pivotal events, bounded contexts, core domain,
  przepływy procesów (pełna gramatyka: aktor → komenda → zdarzenie → polityka)
  oraz relacje między kontekstami. Zapisuje wynik do state/phase-2-output.md.
tools: Read, Write, Bash, LS
---

# Deep Analysis — Process Level (Skill 2)

## Zasada nadrzędna

Process Level = zejście w głąb wybranych obszarów z Big Picture.
Celem jest zrozumienie jak procesy działają, kto za co odpowiada i gdzie są granice kontekstów.

Pełna gramatyka tej fazy (Brandolini, "the picture that explains everything"):

```
Read Model → Aktor → Komenda → System/BC → Zdarzenie → Polityka → Komenda → ...
```

Nie modelujemy jeszcze agregatów, niezmienników ani klas — to Design Level.

**Rozróżnienie kluczowe (Khononov, Evans):**
- **Subdomena** = przestrzeń problemu — odkrywamy ją, istnieje niezależnie od nas
- **Bounded Context** = przestrzeń rozwiązania — projektujemy go, to decyzja
- Typologia core / supporting / generic dotyczy **subdomen**, nie BC
- Punkt startu: 1 subdomena ≈ 1 BC; odstępstwa tylko świadome i uzasadnione

---

## Dane wejściowe od orkiestratora

```
- tryb pracy (1 / 2 / 3 / 4)
- tryb analizy (audit / elicit) + input-quality (z bramy B2 orkiestratora)
- state/inputs/processed/phase-2-seed.md (jeśli istnieje)
- state/phase-1-output.md (wymagany)
- z handoffu Fazy 1: pivotal_events (PE-IDs), bleed_through, hotspots
- otwarte pytania z Fazy 1 (lista blokerów)
- ścieżka do katalogu state/
```

Tryb pracy wpływa na sposób zadawania pytań:
- Tryb 1: agenci odpytują konteksty, użytkownik weryfikuje syntezę
- Tryb 2 (domyślny): pytania kierowane do użytkownika, jedno na raz
- Tryb 3/4: zgodnie z podziałem ustalonym przez orkiestratora

Tryb analizy `audit`: nie pytaj o to, co wsad i Faza 1 już zawierają — weryfikuj
i pytaj wyłącznie o luki. Każde pytanie powinno wynikać z konkretnego braku
(heurystyka, która go wykryła), nie z procedury zbierania od zera.

---

## Krok 0 — Utwórz strukturę katalogów

```
state/
  phase-2/
    working/     ← pliki robocze zbierane w trakcie sesji
```

---

## Krok 1 — Załaduj kontekst

Wczytaj `state/phase-1-output.md`. Wyświetl użytkownikowi:
- listę zdarzeń domenowych z Fazy 1
- pivotal events z Fazy 1 (PE-IDs)
- listę hot spotów z Fazy 1
- otwarte pytania oznaczone jako blokery

Jeśli istnieje `state/inputs/processed/phase-2-seed.md` — wczytaj i uwzględnij.

Zapytaj: "Od której grupy zdarzeń z Fazy 1 zaczynamy — która część procesu jest
najważniejsza albo najbardziej niejasna?"

**Hot spoty:** od tego momentu każdą niejasność, spór lub sprzeczność zapisuj na bieżąco
do `state/phase-2/working/hotspots.md` (format: `HS2-NNN | opis | źródło | status: open/resolved`).
Hot spoty z Fazy 1, które ta faza rozstrzyga, oznacz jako resolved z odwołaniem.

---

## Krok 1a — Zakres fazy (bleed-through)

Zanim zaczniesz pytania merytoryczne, potwierdź zakres:

```
Process Level bazowe (domyślne, zgodne z Brandolinim):
  subdomeny + typologia, pivotal events, bounded contexts, core domain,
  przepływy (aktor → komenda → zdarzenie), polityki, read modele, context map

Opcjonalnie można dodać (zapytaj czy chcesz):
  [+] Wstępne reguły biznesowe na poziomie przepływu
      ("co musi być prawdą, żeby X nastąpiło" — bez przypisania do agregatu)

Nie wnosimy na tym etapie:
  [–] Agregaty i niezmienniki (Design Level)
  [–] Szczegóły implementacji
```

---

## Krok 2 — Odkrycie subdomen

Na podstawie zdarzeń i aktorów z Fazy 1 szukaj kandydatów na subdomeny.
Subdomeny odkrywamy pytaniami, nie zakładamy ich z góry.

**Heurystyki odkrywania:**

**H1 — Językowa (Evans, ubiquitous language):**
```
Czy różne grupy używają różnych słów na ten sam byt?
Czy to samo słowo znaczy co innego dla różnych grup?
(np. "zgłoszenie" dla zawodnika vs "zgłoszenie" dla serwisu rowerowego)
→ zmiana języka = sygnał granicy
```

**H2 — Właścicielska:**
```
Kto decyduje, gdy w tej części procesu coś idzie nie tak?
Gdzie zmiana jednej reguły nie powinna wymuszać zmiany w innej części?
→ inny decydent = inna subdomena
```

**H3 — Zmienności (analogia do osi zmian, Martin):**
```
Co zmienia się często, co rzadko? Co zmienia się z innych powodów niż reszta?
→ co zmienia się razem i z tego samego powodu, należy do tej samej subdomeny
```

**H4 — Cięcia:**
```
Co dałoby się wyłączyć lub zastąpić zewnętrznym systemem bez wpływu na resztę?
Które części mogłyby działać jako oddzielny zespół?
→ naturalne cięcie = kandydat na granicę
```

Dla każdego kandydata: nazwa + odpowiedzialność (jedno zdanie) + zdarzenia z Fazy 1,
które do niego należą.

Zapisz do `state/phase-2/working/subdomains.md`.

---

## Krok 3 — Typologia subdomen (core / supporting / generic)

Dla każdej subdomeny określ typ. Klasyfikacja jest **per firma** i zmienia się w czasie
(płatności: generic dla sklepu, core dla operatora płatności).

**Macierz klasyfikacyjna (Khononov, "Learning DDD"):**

| Typ | Złożoność logiki | Wyróżnia na rynku? | Strategia |
|---|---|---|---|
| Core | wysoka | tak | budujemy sami, najlepsi ludzie, rich domain model |
| Supporting | niska | nie, ale konieczne | budujemy prosto, CRUD jest OK, można outsource |
| Generic | wysoka, ale rozwiązana | nie | kupujemy gotowe / SaaS |

**Heurystyki rozstrzygające:**

**T1 — Test kup-czy-buduj:**
```
Czy istnieje gotowy produkt, który to robi wystarczająco dobrze?
Tak, i nie tracimy przewagi → generic
Nie ma gotowego, ale logika jest prosta → supporting
Nie ma gotowego i logika jest złożona → kandydat na core
```

**T2 — Test najtańszego rozwiązania (Khononov):**
```
Czy najtańsze działające rozwiązanie wystarczy biznesowi?
Tak → to nie jest core
Nie — przewaga wymaga, żeby było lepsze niż u konkurencji → core
```

**T3 — Test zmienności:**
```
Czy ta subdomena ma stan "gotowe" — czy będzie ewoluować bez końca?
Core nie ma stanu końcowego; generic i supporting stabilizują się.
```

**T4 — Test kosztu błędu:**
```
Gdzie błąd jest najdroższym błędem biznesowym?
Gdzie biznes inwestuje najwięcej uwagi i wiedzy eksperckiej?
→ tam zwykle jest core
```

**Pułapki:**
- core nie musi być największą częścią systemu — często jest mała
- "centralna tabela w bazie" ≠ core domain
- jeden system może mieć więcej niż jedną subdomenę core, ale rzadko — kwestionuj

Format zapisu: nazwa → typ → uzasadnienie (które testy zdecydowały).
Core domain wskaż wprost z uzasadnieniem.

Zapisz do `state/phase-2/working/subdomain-types.md`.

---

## Krok 4 — Pivotal Events (weryfikacja i uzupełnienie)

Pivotal events to zdarzenia zmieniające stan domeny nieodwracalnie lub wyraźnie
rozdzielające fazy procesu. To naturalni kandydaci na granice BC (Brandolini).

**Nie odkrywaj od zera** — Faza 1 zebrała już pivotal events (PE-IDs z handoffu).

1. Wczytaj PE z Fazy 1 i przedstaw je w kontekście subdomen z Kroku 2:
   ```
   "Czy te punkty zwrotne nadal pasują, gdy patrzymy na granice subdomen?
    Czy któryś z nich przestał być pivotal po doprecyzowaniu procesu?"
   ```
2. Uzupełnij nowe pivotal events odkryte na tym poziomie szczegółowości:
   ```
   1. Które zdarzenia są punktem bez powrotu — po nich coś zmienia się na zawsze?
   2. Które zdarzenia kończą jedną fazę i zaczynają kolejną?
   3. Po którym zdarzeniu odpowiedzialność przejmuje inny zespół / system?
   4. Które zdarzenia są widoczne dla klienta lub zewnętrznego obserwatora?
   ```
3. Numeracja `PE-` jest kontynuowana z Fazy 1 (nowe dostają kolejne numery).
   PE zdegradowane oznacz `[zdegradowany: powód]` — nie usuwaj.
   Jeśli degradacja zmienia wnioski Fazy 1 → adnotacja retrospektywna w phase-1-output.md.

Zapisz do `state/phase-2/working/pivotal-events.md` (z oznaczeniem źródła: Faza 1 / Faza 2).

---

## Krok 5 — Bounded Contexts

Wyprowadź BC z subdomen (Krok 2–3) i pivotal events (Krok 4).
Punkt startu: 1 subdomena = 1 BC. Każde odstępstwo zapisz z uzasadnieniem.

**Heurystyki wyznaczania granic BC:**

**B1 — Jedno znaczenie (Evans):**
```
W obrębie jednego BC każdy termin ma dokładnie jedno znaczenie.
Jeśli "oferta" znaczy dwie rzeczy → dwa konteksty albo dwa pojęcia.
```

**B2 — Pivotal events jako szwy:**
```
Granica BC często przebiega na pivotal event:
przed zdarzeniem — język i model kontekstu A, po — kontekstu B.
```

**B3 — Spójność transakcyjna:**
```
Co musi być spójne natychmiast (w jednej transakcji)? → ten sam BC
Co może być spójne ostatecznie (eventual consistency)? → może iść przez granicę
```

**B4 — Własność danych:**
```
Każda dana ma jednego właściciela (jeden kontekst pisze, inni czytają kopie).
Dwa konteksty walczące o zapis tego samego bytu → granica źle postawiona
albo byt trzeba rozdzielić na dwa pojęcia (B1).
```

**B5 — Zespołowa (Conway; Skelton & Pais, "Team Topologies"):**
```
Jeden BC ≤ jeden zespół. BC większy niż cognitive load zespołu → podziel.
Dwa zespoły w jednym BC → konflikt modeli, kwestionuj granicę.
```

**Antywzorce:**
- BC wokół encji ("kontekst Klienta") zamiast wokół zdolności biznesowej
- BC = warstwa techniczna (UI-context, DB-context)

Dla każdego BC:
- nazwa
- odpowiedzialność (jedno zdanie)
- subdomena źródłowa + typ (z Kroku 3)
- zdarzenia z Fazy 1 należące do tego BC
- pivotal events na granicach

Zapisz do `state/phase-2/working/bounded-contexts.md`.

---

## Krok 6 — Przepływy procesów

Dla każdego BC zbierz przepływ w pełnej gramatyce Process Level:

**Jeśli Faza 1 zbierała read modele (`bleed_through.read_models == true` w handoffie):**
wczytaj sekcję Read Modele z phase-1-output.md jako punkt startowy — weryfikuj
i uzupełniaj, nie pytaj od zera o to co już zebrano.

```
"Kto (aktor) i na podstawie czego (read model) podejmuje decyzję?
 Jaką komendę wydaje? Co ją obsługuje? Jakie zdarzenie powstaje?
 Jakie reguły biznesowe obowiązują? Co może pójść nie tak?"
```

Na każdy proces:
- wyzwalacz (aktor / czas / zdarzenie z innego BC / system zewnętrzny)
- kroki numerowane w formacie: `[Aktor] --[Komenda]--> [BC/System] ==> [Zdarzenie]`
- read modele: co aktor musi widzieć, żeby podjąć decyzję
- reguły biznesowe
- wyjątki i ścieżki alternatywne
- przepływ odwrotny: błędy, kompensacje, cofnięcia

**Weryfikacja odwrotną narracją (reverse narrative, Brandolini):**
Po spisaniu przepływu przejdź od zdarzenia końcowego wstecz:

```
"Żeby [zdarzenie końcowe] mogło nastąpić — co musiało się wydarzyć wcześniej?
 Skąd wzięły się potrzebne dane? Kto podjął decyzję i na jakiej podstawie?"
```

Każda luka znaleziona wstecz = brakujące zdarzenie, komenda lub read model.

Zapisz do `state/phase-2/working/processes/[nazwa-procesu].md`.

---

## Krok 7 — Polityki (Event → Policy → Command)

Zidentyfikuj polityki łączące zdarzenia z akcjami:

```
"Gdy [zdarzenie] — co się dzieje automatycznie?
 Czy są reguły które mówią: zawsze gdy X, to Y?
 Kto dziś tego pilnuje — system czy człowiek z czeklistą?"
```

Format: `Gdy [E-ID] → [reguła] → [komenda]`

Dla każdej polityki oznacz:
- automatyczna (system) czy ludzka (procedura, nawyk, "Grażyna pamięta")
- ludzkie polityki to ukryta logika procesu — częsty hot spot, zapisz HS2 jeśli krucha

Polityki przekraczające granicę BC oznacz — to wejście do context map (Krok 8).

Zapisz do `state/phase-2/working/policies.md`.

---

## Krok 8 — Relacje między kontekstami (context map)

```
"Które konteksty muszą ze sobą rozmawiać?
 Kto jest właścicielem danych? Kto konsumuje dane innych?
 Kto może żądać zmian od kogo?"
```

**Katalog relacji (Evans, DDD; Vernon, IDDD):**

| Wzorzec | Kiedy |
|---|---|
| Partnership | dwa zespoły zależne wzajemnie, wspólny cel, synchronizują plany |
| Shared Kernel | współdzielony fragment modelu; wymaga dyscypliny — ryzykowny |
| Customer–Supplier | downstream może negocjować wymagania, upstream je planuje |
| Conformist | downstream przyjmuje model upstream bez tłumaczenia |
| Anticorruption Layer (ACL) | downstream tłumaczy obcy model na własny język |
| Open Host Service (OHS) | upstream wystawia stabilne API dla wielu konsumentów |
| Published Language | wspólny, udokumentowany format wymiany (zwykle z OHS) |
| Separate Ways | integracja droższa niż jej brak — brak relacji |
| Big Ball of Mud | granica wokół bałaganu, którego nie modelujemy — tylko ACL od niego |

**Heurystyki doboru relacji:**

**R1 — Układ sił:**
```
Kto może żądać zmian od kogo?
Upstream nie słucha downstream → Conformist albo ACL.
Upstream słucha i planuje pod downstream → Customer–Supplier.
Obaj zależni wzajemnie → Partnership.
```

**R2 — Jakość obcego modelu:**
```
Czy model upstream pasuje do naszego języka?
Pasuje i jest stabilny → Conformist (tanio).
Nie pasuje albo to legacy/system zewnętrzny → ACL.
```

**R3 — Ochrona core (Vernon):**
```
Core domain nigdy nie jest Conformistem.
Między core a legacy / systemem zewnętrznym → zawsze ACL.
```

**R4 — Liczba konsumentów:**
```
Jeden konsument → relacja punktowa (CS / Conformist / ACL).
Wielu konsumentów → OHS + Published Language.
```

**R5 — Koszt integracji:**
```
Czy koordynacja zespołów kosztuje więcej niż duplikacja?
Tak → Separate Ways.
```

Dla każdej zależności:
- kierunek (upstream / downstream)
- wzorzec z katalogu + heurystyka, która zdecydowała
- co jest przekazywane (zdarzenie integracyjne, DTO, API)

Zapisz do `state/phase-2/working/context-map.md`.

---

## Krok 9 — Synteza i zapis wyniku

Wczytaj `templates/phase-2-template.md`.
Wypełnij szablon danymi z plików working/:

```
working/subdomains.md         → sekcja Subdomeny
working/subdomain-types.md    → sekcja Typologia i Core Domain
working/pivotal-events.md     → sekcja Pivotal Events
working/bounded-contexts.md   → sekcja Bounded Contexts
working/processes/            → sekcja Przepływy procesów
working/policies.md           → sekcja Polityki
working/context-map.md        → sekcja Relacje między kontekstami
working/hotspots.md           → sekcja Hot spoty
```

Wygeneruj ID dla każdego elementu:
- `SD-` subdomeny, `BC-` bounded contexts, `P-` procesy, `RB-` reguły, `EX-` wyjątki,
  `POL-` polityki, `D-` relacje, `HS2-` hot spoty tej fazy
- `PE-` pivotal events: **kontynuacja numeracji z Fazy 1**, nie nadawaj od nowa
Wypełnij frontmatter: `input-quality` i `lossy-files` z bramy B2 (`n/a` w trybie elicit).
Wypełnij sekcję Wnioski i Otwarte pytania.
**Reguła pewności:** wniosek oparty wyłącznie na materiale `lossy` → pewność maksymalnie `średnia`.

Jeśli szablon nie ma sekcji Subdomeny / Pivotal Events / Hot spoty — dodaj je
i zgłoś orkiestratorowi potrzebę aktualizacji szablonu.

Zapisz do `state/phase-2-output.md`.

---

## Krok 10 — Weryfikacja z użytkownikiem

Pytania spójnościowe przed podsumowaniem:

```
1. Czy każda subdomena ma typ, a każdy BC właściciela?
2. Czy każdy pivotal event jest osiągalny — istnieje ścieżka, która do niego prowadzi?
3. Czy nie ma wysp — BC bez relacji z resztą?
4. Czy nie ma duplikatów — to samo pojęcie w dwóch BC bez uzgodnienia (narusza B1/B4)?
5. Czy każdy przepływ ma koniec — co jest ostatecznym zdarzeniem?
```

Wyświetl podsumowanie:
```
Zebrano:
- Subdomeny: [N] (Core: [N], Supporting: [N], Generic: [N])
- Core Domain: [nazwa]
- Pivotal Events: [N]
- Bounded Contexts: [N]
- Przepływów procesów: [N]
- Polityk: [N] (w tym ludzkich/nieautomatycznych: [N])
- Relacji między BC: [N]
- Hot spotów: nowych [N], rozwiązanych z Fazy 1 [N]
- Blokerów: [N]
```

Zapytaj: "Czy model odzwierciedla Twoją wiedzę? Co wymaga korekty?"
Nanieś korekty i zapisz zaktualizowany `state/phase-2-output.md`.

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "process-level",
  "status": "completed",
  "output_file": "state/phase-2-output.md",
  "bounded_contexts": [
    {"id": "BC-001", "name": "nazwa", "type": "core|supporting|generic"}
  ],
  "core_domain": {"id": "SD-001", "name": "nazwa subdomeny"},
  "open_questions": ["lista pytań z phase-2-output.md gdzie priorytet = blocker"],
  "suggested_next": "design-level",
  "user_decision_required": true
}
```

`bounded_contexts` i `core_domain` są wymagane przez orkiestrator do weryfikacji
warunku przejścia Process Level → Design Level oraz jako wejście Skill 3.
