---
name: deep-analysis-big-picture
description: >
  Prowadzi fazę Big Picture Event Stormingu. Uruchamiaj jako Skill 1 gdy orkiestrator
  zleca analizę Big Picture. Zbiera zdarzenia domenowe, aktorów, granice systemu i hot spoty.
  Obsługuje Tryby 1-4. Zapisuje wynik do state/phase-1-output.md.
  Zwraca handoff z otwartymi pytaniami i sugestią przejścia do Process Level.
tools: Read, Write, Bash, LS
---

# Deep Analysis — Big Picture (Skill 1)

## Zasada nadrzędna

Big Picture = szeroki obraz domeny bez zagłębiania się w szczegóły implementacyjne.
Celem jest zebranie zdarzeń domenowych, aktorów i granic — nie ich analiza.
Jeśli rozmowa schodzi w szczegóły procesu → zanotuj jako hot spot, wróć do szerokiego obrazu.

---

## Dane wejściowe od orkiestratora

```
- tryb pracy (1 / 2 / 3 / 4)
- cel analizy
- state/inputs/processed/phase-1-seed.md (jeśli istnieje)
- ścieżka do katalogu state/
```

---

## Krok 0 — Utwórz strukturę katalogów

```
state/
  phase-1/
    working/     ← pliki robocze zbierane w trakcie sesji
```

---

## Krok 1 — Załaduj kontekst

Jeśli istnieje `state/inputs/processed/phase-1-seed.md`:
- wczytaj i wyświetl użytkownikowi podsumowanie: ile zdarzeń / aktorów / hot spotów już jest
- zapytaj: "Chcesz zacząć od tych danych i je uzupełnić, czy zacząć od zera?"

Jeśli seed nie istnieje — zacznij od zera.

---

## Krok 2 — Zbieranie zdarzeń domenowych

### Tryb 2 (domyślny) — pytania do użytkownika

Zbieraj zdarzenia iteracyjnie. Na każdą rundę:

```
"Wymień zdarzenia które dzieją się w systemie — co się wydarzyło?
 Zapisuj w czasie przeszłym (np. 'Zamówienie złożone', 'Płatność odrzucona').
 Gdy skończysz daną część domeny — powiedz 'dalej' lub wskaż kolejny obszar."
```

Po każdej rundzie:
- zapisz zebrane zdarzenia do `state/phase-1/working/events.md`
- wyświetl aktualną listę i zapytaj czy coś brakuje

Kontynuuj aż użytkownik powie "gotowe" lub nie doda nowych zdarzeń przez dwie rundy.

### Tryb 1 (agentowy)

Na podstawie seed file i opisu systemu zaproponuj listę zdarzeń domenowych.
Wyświetl użytkownikowi do weryfikacji i korekty.

---

## Krok 2b — Eksploracja poza materiałami (TODO)

> **NOTATKA — do zaimplementowania**
>
> Po zebraniu zdarzeń z materiałów wsadowych agent powinien aktywnie wychodzić poza kontekst.
> Big Picture ma dwa tryby poznawcze jednocześnie:
> - eksploatacja — wydobycie tego co ekspert domenowy wie
> - eksploracja — zasugerowanie tego czego ekspert nie widzi bo jest za blisko domeny
>
> Implementacja:
> - na podstawie zebranych zdarzeń i opisu domeny zaproponuj zdarzenia typowe dla podobnych domen
>   których nie ma w materiałach
> - zaproponuj aktorów których logicznie powinni istnieć a nie pojawili się w materiale
> - wskaż obszary domeny niepokryte żadnym zdarzeniem
>
> Każda propozycja oznaczona jako `suggested — not in source materials`.
> Użytkownik potwierdza / odrzuca / modyfikuje.
> Zaakceptowane propozycje wchodzą do events.md z tagiem `discovered`.
>
> **Test weryfikacyjny:**
> Dostarcz materiały z celowo wyciętym obszarem domeny który powinien istnieć.
> Sprawdź czy agent: zauważy lukę, zaproponuje brakujący obszar, oznaczy go poprawnie.
> Process Level i Design Level NIE stosują tego kroku — tam wychodzenie poza kontekst
> wprowadza niespójności.

---

## Krok 3 — Identyfikacja aktorów

```
"Kto lub co wywołuje te zdarzenia?
 Może być człowiek, system zewnętrzny, harmonogram (czas), inny system."
```

Przypisz każdego aktora do zdarzeń z Kroku 2.
Zapisz do `state/phase-1/working/actors.md`.

---

## Krok 4 — Granice systemu

```
"Co jest w systemie a co poza nim?
 Które systemy zewnętrzne wchodzą w interakcję i w jaki sposób?"
```

Zapisz do `state/phase-1/working/boundaries.md`.

---

## Krok 5 — Hot Spoty

Podczas Kroków 2-4 oznaczaj hot spoty na bieżąco.
Hot spot = miejsce gdzie:
- użytkownik się waha lub mówi "to zależy"
- pojawia się konflikt między obszarami
- brakuje wiedzy domenowej
- zdarzenie wywołuje wiele pytań

Na końcu tego kroku zapytaj:
```
"Czy są miejsca w domenie które budzą wątpliwości,
 są niejasne lub wymagają decyzji biznesowej?"
```

Zapisz do `state/phase-1/working/hotspots.md`.

---

## Krok 6 — Synteza i zapis wyniku

Wczytaj szablony: `templates/phase-1-template.md`.
Wypełnij szablon danymi z plików working/:

```
state/phase-1/working/events.md    → sekcja Zdarzenia domenowe
state/phase-1/working/actors.md    → sekcja Aktorzy
state/phase-1/working/boundaries.md → sekcja Granice systemu
state/phase-1/working/hotspots.md  → sekcja Hot Spoty
```

Wygeneruj ID dla każdego elementu (A-001, E-001, H-001).
Wypełnij sekcję Wnioski — minimum 3 wnioski z oceną pewności.
Wypełnij sekcję Otwarte pytania — oznacz blokery.

Zapisz do `state/phase-1-output.md`.

---

## Krok 7 — Weryfikacja z użytkownikiem

Wyświetl podsumowanie:
```
Zebrano:
- Aktorów: [N]
- Zdarzeń domenowych: [N]
- Granic / integracji: [N]
- Hot Spotów: [N]
- Blokerów: [N]
```

Zapytaj: "Czy wynik odzwierciedla Twoją wiedzę o domenie? Co wymaga korekty?"
Nanieś korekty i zapisz zaktualizowany `state/phase-1-output.md`.

---

## Zwróć handoff do orkiestratora

```json
{
  "phase": "big-picture",
  "status": "completed",
  "output_file": "state/phase-1-output.md",
  "open_questions": ["lista pytań z phase-1-output.md gdzie priorytet = blocker"],
  "suggested_next": "process-level",
  "user_decision_required": true
}
```
