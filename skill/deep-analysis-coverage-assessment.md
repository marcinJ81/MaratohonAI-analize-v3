---
name: deep-analysis-coverage-assessment
description: >
  Sub-skill oceny pokrycia faz analizy materiałami wejściowymi.
  Uruchamiaj po etapie 'prepare' skilla data-prep, przed etapem 'seed' i przed uruchomieniem
  pierwszej fazy. Zwraca ocenę % pokrycia per faza i rekomendacje dla orchestratora.
tools: Read, Write, Bash
---

# Deep Analysis — Coverage Assessment (Sub-Skill)

## Odpowiedzialność

Ten skill robi dokładnie jedno: ocenia w jakim stopniu dostarczone materiały pokrywają
wymagania danych dla każdej fazy analizy.

Nie analizuje domeny. Nie zadaje pytań domenowych.
Pyta tylko gdy brakuje danych do samej oceny pokrycia.

---

## Wejście

```
state/inputs/processed/          ← znormalizowane materiały (*.md)
[złożoność systemu]              ← niska / średnia / wysoka (z handoffu data-prep)
[tryb analizy]                   ← audit / elicit (z orchestratora)
```

---

## Kryteria pokrycia per faza

### Big Picture — kryteria oceny

| Kryterium | Waga | Jak oceniać |
|---|---|---|
| Aktorzy zidentyfikowani | 25% | Czy materiały wymieniają kto korzysta z systemu? |
| Zdarzenia domenowe | 30% | Czy istnieje lista zdarzeń? Czy jest kompletna względem złożoności? |
| Granice systemu | 25% | Czy wiadomo co jest w systemie, co na zewnątrz? |
| Integracje zewnętrzne | 20% | Czy wymieniono systemy zewnętrzne z którymi system się łączy? |

Liczba zdarzeń względem złożoności:
- niska: 5–15 zdarzeń = pełne pokrycie
- średnia: 15–40 zdarzeń = pełne pokrycie
- wysoka: 40+ zdarzeń = pełne pokrycie

### Process Level — kryteria oceny

Zakres fazy zgodny z `deep-analysis-process-level.md`: subdomeny + typologia,
pivotal events, bounded contexts, przepływy w pełnej gramatyce
(aktor → komenda → zdarzenie), polityki, relacje między kontekstami, hot spoty.

| Kryterium | Waga | Jak oceniać |
|---|---|---|
| Przepływy procesów | 25% | Czy materiały opisują kroki procesów: kto (aktor) wykonuje jaką akcję (komendę) i z jakim skutkiem (zdarzeniem)? Same listy kroków bez aktorów = 0.5 |
| Subdomeny i core domain | 20% | Czy da się wskazać obszary odpowiedzialności i ich wagę biznesową? Czy materiały mówią co jest przewagą konkurencyjną? |
| Reguły biznesowe i polityki | 20% | Czy są wymienione warunki, ograniczenia, reakcje na zdarzenia ("gdy X, wtedy Y")? |
| Granice kontekstów i pivotal events | 15% | Czy widać naturalne granice odpowiedzialności? Czy materiały wskazują punkty zwrotne procesu (zdarzenia bez powrotu, zmiany właściciela)? |
| Relacje między kontekstami | 10% | Czy opisano kto z kim się integruje, kto jest właścicielem danych, co jest przekazywane? |
| Wyjątki i hot spoty | 10% | Czy są opisane przypadki brzegowe, konflikty, ścieżki kompensacji? |

### Design Level — kryteria oceny

Zakres fazy: agregaty, niezmienniki, kontrakty per bounded context.
Komendy i polityki oceniane są w Process Level (gramatyka przepływu), nie tutaj.

| Kryterium | Waga | Jak oceniać |
|---|---|---|
| Agregaty / encje | 30% | Czy materiały opisują struktury danych lub modele? |
| Reguły niezmienników | 25% | Czy są reguły spójności danych / inwarianty — co nigdy nie może być naruszone? |
| Kontrakty / interfejsy | 25% | Czy są opisy API, zdarzeń publikowanych na zewnątrz kontekstu, formatów wymiany? |
| Reguły walidacji | 20% | Czy są reguły walidacji z rozróżnieniem źródła (domena vs aplikacja)? |

---

## Algorytm oceny

Dla każdej fazy:

1. Dla każdego kryterium przyznaj punkty: `0 / 0.5 / 1.0`
   - `0` — brak danych
   - `0.5` — dane częściowe lub domniemane
   - `1.0` — dane obecne i wystarczające

2. Oblicz wynik ważony: `suma(punkty × waga)` → wynik w %

3. Sklasyfikuj wynik:
   - `≥ 75%` — **wystarczające** — faza może startować
   - `40–74%` — **częściowe** — faza może startować z ograniczeniami
   - `< 40%` — **niewystarczające** — rekomendowane uzupełnienie przed startem

---

## Wyjście

### Plik: `state/inputs/processed/index.md`

```markdown
# Inputs Index

## Materiały źródłowe
| Plik | Typ | Konwersja | Użyty w fazach |
|---|---|---|---|
| [nazwa] | [typ] | clean / lossy | phase-1, phase-2 |

## Ocena pokrycia

| Faza | Pokrycie | Status | Uwagi |
|---|---|---|---|
| Big Picture | [X%] | wystarczające / częściowe / niewystarczające | [co brakuje] |
| Process Level | [X%] | wystarczające / częściowe / niewystarczające | [co brakuje] |
| Design Level | [X%] | wystarczające / częściowe / niewystarczające | [co brakuje] |

## Szczegóły per faza

### Big Picture
| Kryterium | Punkty | Uwaga |
|---|---|---|
| Aktorzy | 0 / 0.5 / 1.0 | |
| Zdarzenia domenowe | 0 / 0.5 / 1.0 | |
| Granice systemu | 0 / 0.5 / 1.0 | |
| Integracje zewnętrzne | 0 / 0.5 / 1.0 | |

### Process Level
| Kryterium | Punkty | Uwaga |
|---|---|---|
| Przepływy procesów | 0 / 0.5 / 1.0 | |
| Subdomeny i core domain | 0 / 0.5 / 1.0 | |
| Reguły biznesowe i polityki | 0 / 0.5 / 1.0 | |
| Granice kontekstów i pivotal events | 0 / 0.5 / 1.0 | |
| Relacje między kontekstami | 0 / 0.5 / 1.0 | |
| Wyjątki i hot spoty | 0 / 0.5 / 1.0 | |

### Design Level
| Kryterium | Punkty | Uwaga |
|---|---|---|
| Agregaty / encje | 0 / 0.5 / 1.0 | |
| Reguły niezmienników | 0 / 0.5 / 1.0 | |
| Kontrakty / interfejsy | 0 / 0.5 / 1.0 | |
| Reguły walidacji | 0 / 0.5 / 1.0 | |

## Rekomendacje dla orchestratora
- [lista brakujących danych z priorytetem: blocker / nice-to-have]
```

### Handoff JSON

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

`user_decision_required: true` tylko gdy ocena jest niejednoznaczna i wymaga kontekstu
którego ten skill nie może pobrać z materiałów.

---

## Reguły działania

- Nie zadawaj pytań domenowych — to nie jest rola tego skilla.
- Pytaj tylko o brakujące dane wejściowe do samej oceny (np. złożoność systemu jeśli orchestrator jej nie przekazał).
- Jeśli konwersja materiału była oznaczona jako `lossy` — obniż ocenę pokrycia tego materiału o 20%.
- Jeśli ten sam obszar jest pokryty przez wiele plików — sumuj pokrycie, nie mnóż.
- Zachowaj ostrożność: lepiej niedoszacować pokrycie niż przeszacować.
- Gotowy Event Storming (zdjęcie tablicy, eksport z Miro) zwykle pokrywa zdarzenia
  i przepływy, ale rzadko typologię subdomen i relacje między kontekstami — oceniaj
  te kryteria osobno, nie zakładaj pokrycia z samego faktu istnienia ES.
- **Tryb audit — twarde blokery (minimalny kontrakt wsadu).** Każdy niespełniony warunek
  zwracaj jako pozycję w `blockers` — orkiestrator zatrzyma na nich bramę akceptacji (B2):
  - brak opisu systemu w materiałach lub seedzie
  - liczba zdarzeń poniżej dolnego progu dla ocenionej złożoności
    (niska < 5, średnia < 15, wysoka < 40)
  - zero zidentyfikowanych integracji zewnętrznych — formułuj jako pytanie:
    "nie znaleziono integracji — czy system działa w izolacji, czy nie ujęto ich we wsadzie?"
  Blokery formułuj jako pytania do interesariuszy, nie jako twierdzenia o brakach —
  luka może być stratą konwersji, nie faktem domenowym.

---

## Integracja z orchestratorem

Orchestrator wywołuje ten skill pomiędzy etapami data-prep:

```
data-prep, etap prepare (składowanie, konwersja, zasilanie wsteczne)
  ↓
[ten skill]: ocena pokrycia → index.md + handoff JSON
  ↓
data-prep, etap seed (kompilacja seed files, coverage z index.md)
```

Pliki derived z zasilania wstecznego istnieją już w `state/inputs/processed/`
w momencie oceny — uwzględniaj je jak każdy inny materiał (jeden przebieg oceny).

`index.md` jest własnością tego skilla — żaden inny skill do niego nie zapisuje.
Lista materiałów i flagi konwersji (`lossy`) pochodzą z `manifest.md` (własność data-prep)
oraz z frontmatter plików `*-processed.md`.

Orchestrator odczytuje `coverage` z handoff JSON i:
- wyświetla użytkownikowi tabelę pokrycia z `index.md`
- przekazuje `recommended_start_phase` jako sugestię (nie decyzję)
- uwzględnia `blockers` przy pytaniu o tryb pracy
