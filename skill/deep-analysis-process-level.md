---
name: deep-analysis-process-level
description: >
  Prowadzi fazę Process Level Event Stormingu. Uruchamiaj jako Skill 2 gdy orkiestrator
  zleca analizę Process Level. Wymaga ukończonego Big Picture (phase-1-output.md).
  Identyfikuje bounded contexts, core domain, przepływy procesów i polityki.
  Zapisuje wynik do state/phase-2-output.md.
tools: Read, Write, Bash, LS
---

# Deep Analysis — Process Level (Skill 2)

## Zasada nadrzędna

Process Level = zejście w głąb wybranych obszarów z Big Picture.
Celem jest zrozumienie jak procesy działają, kto za co odpowiada i gdzie są granice kontekstów.
Nie modelujemy jeszcze klas ani agregatów — to jest Design Level.

---

## Dane wejściowe od orkiestratora

```
- tryb pracy (1 / 2 / 3 / 4)
- state/inputs/processed/phase-2-seed.md (jeśli istnieje)
- state/phase-1-output.md (wymagany)
- otwarte pytania z Fazy 1 (lista blokerów)
- ścieżka do katalogu state/
```

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
- listę hot spotów z Fazy 1
- otwarte pytania oznaczone jako blokery

Jeśli istnieje `state/inputs/processed/phase-2-seed.md` — wczytaj i uwzględnij.

Zapytaj: "Od którego obszaru zaczynamy — core domain czy konkretny proces?"

---

## Krok 2 — Identyfikacja Bounded Contexts

Na podstawie zdarzeń z Fazy 1 zaproponuj grupowanie:

```
"Patrząc na zdarzenia — które z nich należą do tego samego obszaru odpowiedzialności?
 Gdzie język się zmienia? Gdzie dane mają inne znaczenie?"
```

Dla każdego kandydata na BC:
- nazwa
- odpowiedzialność (jedno zdanie)
- które zdarzenia z Fazy 1 należą do tego BC

Zapisz do `state/phase-2/working/bounded-contexts.md`.

---

## Krok 3 — Core Domain

```
"Który z tych kontekstów stanowi największą wartość biznesową?
 Co jest unikalną przewagą systemu — czego konkurencja nie może łatwo skopiować?"
```

Zapisz nazwę i uzasadnienie do `state/phase-2/working/core-domain.md`.

---

## Krok 4 — Przepływy procesów

Dla każdego BC z Kroku 2 zbierz przepływ:

```
"Jak wygląda krok po kroku proces w kontekście [BC]?
 Co go wyzwala? Jakie reguły biznesowe obowiązują?
 Co może pójść nie tak?"
```

Na każdy proces:
- wyzwalacz
- kroki numerowane
- reguły biznesowe
- wyjątki i ścieżki alternatywne

Zapisz do `state/phase-2/working/processes/[nazwa-procesu].md`.

---

## Krok 5 — Polityki (Event → Policy → Command)

Zidentyfikuj polityki łączące zdarzenia z akcjami:

```
"Gdy [zdarzenie] — co się dzieje automatycznie?
 Czy są reguły które mówią: jeśli X to zawsze Y?"
```

Format: `Gdy [E-ID] → [reguła] → [akcja/komenda]`

Zapisz do `state/phase-2/working/policies.md`.

---

## Krok 6 — Zależności między kontekstami

```
"Które konteksty muszą ze sobą rozmawiać?
 Kto jest właścicielem danych? Kto konsumuje dane innych?"
```

Dla każdej zależności:
- kierunek (upstream / downstream)
- typ relacji (ACL, shared kernel, OHS, conformist)
- co jest przekazywane (zdarzenie integracyjne, DTO, API)

Zapisz do `state/phase-2/working/context-map.md`.

---

## Krok 7 — Synteza i zapis wyniku

Wczytaj `templates/phase-2-template.md`.
Wypełnij szablon danymi z plików working/:

```
working/bounded-contexts.md  → sekcja Bounded Contexts
working/core-domain.md       → sekcja Core Domain
working/processes/           → sekcja Przepływy procesów
working/policies.md          → sekcja Polityki
working/context-map.md       → sekcja Zależności między kontekstami
```

Wygeneruj ID dla każdego elementu (BC-001, RB-001, P-001, D-001).
Wypełnij sekcję Wnioski i Otwarte pytania.

Zapisz do `state/phase-2-output.md`.

---

## Krok 8 — Weryfikacja z użytkownikiem

Wyświetl podsumowanie:
```
Zebrano:
- Bounded Contexts: [N] (Core: [N], Supporting: [N], Generic: [N])
- Core Domain: [nazwa]
- Przepływów procesów: [N]
- Polityk: [N]
- Zależności między BC: [N]
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
  "open_questions": ["lista pytań z phase-2-output.md gdzie priorytet = blocker"],
  "suggested_next": "design-level",
  "user_decision_required": true
}
```
