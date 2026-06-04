---
name: deep-analysis-coverage-assessment
description: >
  Sub-skill oceny pokrycia faz analizy materiałami wejściowymi.
  Uruchamiaj po normalizacji materiałów (Krok B orchestratora), przed uruchomieniem
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
[złożoność systemu]              ← niska / średnia / wysoka (z orchestratora)
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

| Kryterium | Waga | Jak oceniać |
|---|---|---|
| Przepływy procesów | 30% | Czy materiały opisują kroki procesów biznesowych? |
| Reguły biznesowe | 30% | Czy są wymienione warunki, ograniczenia, polityki? |
| Wyjątki i hot spoty | 20% | Czy są opisane przypadki brzegowe, konflikty? |
| Bounded Contexts (wstępne) | 20% | Czy dają się wyróżnić naturalne granice odpowiedzialności? |

### Design Level — kryteria oceny

| Kryterium | Waga | Jak oceniać |
|---|---|---|
| Agregaty / encje | 30% | Czy materiały opisują struktury danych lub modele? |
| Kontrakty / interfejsy | 25% | Czy są opisy API, zdarzeń, komend? |
| Bounded Contexts (nazwane) | 25% | Czy konteksty są explicite zdefiniowane? |
| Reguły niezmienników | 20% | Czy są reguły spójności danych / inwarianty? |

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
[analogicznie]

### Design Level
[analogicznie]

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
```

---

## Integracja z orchestratorem

Orchestrator wywołuje ten skill po Kroku B (normalizacja), przed Krokiem E (data-prep).

```
Krok B: normalizacja materiałów
  ↓
[ten skill]: ocena pokrycia → index.md + handoff JSON
  ↓
Krok D: zasilanie wsteczne (jeśli potrzebne)
  ↓
Krok E: data-prep seed dla fazy startowej
```

Orchestrator odczytuje `coverage` z handoff JSON i:
- wyświetla użytkownikowi tabelę pokrycia z `index.md`
- przekazuje `recommended_start_phase` jako sugestię (nie decyzję)
- uwzględnia `blockers` przy pytaniu o tryb pracy
