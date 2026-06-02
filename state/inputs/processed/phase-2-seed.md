# Seed: Phase 2 — Process Level
<!-- generated-by: orchestrator, session: maraton-rowerowy -->
<!-- coverage: 60% -->
<!-- sources: all-materials-processed.md, phase-1-output.md, 18 PNG images -->

## Kontekst startu

Analiza systemu obsługi maratonu rowerowego. Big Picture zakończony (80% pokrycia).
Cel tej fazy: zidentyfikować Bounded Contexts, nazwać Core Domain, opisać przepływy procesów z regułami biznesowymi i wyjątkami.

---

## Kandydaci na Bounded Contexts (z Big Picture)

| Robocza nazwa BC    | Krótki opis                                          | Typ (hipoteza)       |
|---------------------|------------------------------------------------------|----------------------|
| Konfiguracja Maratonu | Parametry zawodów, regulamin, terminy              | Supporting / Generic |
| Rejestracja         | Zapisy uczestników, wybór dystansu, zmiana dystansu  | Supporting           |
| Opłaty              | Płatności, niedopłaty, nadpłaty, promocje, zwroty    | Supporting / Generic |
| Grupy Startowe      | Generacja grup, przypisanie numerów, zmiany grup     | Core (?)             |
| Przygotowanie Startu| Weryfikacja, pakiety startowe, kaucja, dyskietki     | Supporting           |
| Start               | Zamknięcie zapisów, start grup i indywidualny        | Core (?)             |
| Pomiar Czasu        | Rejestracja przejazdów, podliczanie, przedłużenia    | Core                 |
| Nakładanie Kar      | Kary za punkty kontrolne i złamanie regulaminu       | Supporting           |
| Zakończenie         | Nagrody, dyplomy, zwrot pakietów                     | Supporting           |
| Wyniki/Ranking      | ⚠️ BRAK W MATERIAŁACH — do omówienia                 | Core (?)             |

---

## Przepływy główne (z materiałów) do pogłębienia

### Przepływ 1: Rejestracja uczestnika
Uczestnik → zarejestrowano uczestnika → wybrano dystans → potwierdzono dane → wyliczono opłatę → [start opłaty]

### Przepływ 2: Opłata za udział
Rozpoczęto opłacenie → przelew bankowy → zarejestrowano przelew → opłacono uczestnictwo  
Wariant: promocja -100% (przez obsługę PK)  
Wariant: niedopłata → opłacenie niedopłaty → opłacono zmianę dystansu  
Wariant: nadpłata → wyliczono kwotę zwrotu → zwrócono nadpłatę  
**⚠️ Hot spot HS-01:** Co gdy generowanie list startuje a uczestnik nie opłacił niedopłaty?

### Przepływ 3: Zmiana dystansu
Uczestnik złożono wniosek → system → zmieniono dystans → wyliczono opłatę → poinformowano  
Reguła: w trakcie opłacania nie można zmienić dystansu

### Przepływ 4: Generacja grup startowych
Automatyczna: system (w regulaminowym terminie) → minimalna liczba uczestników z numerami → wygenerowano grupy  
Manualna: administrator  
Wariant po terminie: opłacono dystans → przyznano numer → dołączono do grupy

### Przepływ 5: Zmiana grupy startowej
Zawodnik → zmień grupę → [guardy: termin, wolne miejsca, nie zmieniał, ten sam dystans, grupy wygenerowane] → zmieniono grupę / nie udało się  
**⚠️ 3 warianty czasowe:** przed generowaniem / po generowaniu / po terminie

### Przepływ 6: Przygotowanie do startu
Kwalifikacja: obsługa PK → weryfikacja tożsamości / oświadczenie → pakiet startowy / dyskwalifikacja  
Weryfikacja przed startem: lista startowa → zweryfikowano / dyskwalifikacja  
Wyjątek: start indywidualny (wyciągnięcie z grupy)  
Wyjątek: zgubienie dyskietki (kaucja → nowa dyskietka)

### Przepływ 7: Start zawodników
Administrator: wystartowano zawodników → zamknięto zapisy  
Sędzia trasy: wystartowano grupę → zarejestrowano czas startu / zdyskwalifikowano / zakończono przedwcześnie

### Przepływ 8: Pomiar czasu
Standardowy: dyskietka → zarejestrowano pomiar → zawodnik zakończył → podliczono czasy → zakończono dystans  
Wydłużony: zawodnik chce jechać dalej → obsługa PK → przedłużono przejazd → odnotowano dodatkowy pomiar  
Trigger końca: wszyscy dojechali LUB minął czas

### Przepływ 9: Nakładanie kar
Sędzia trasy → nałożono karę (złamanie regulaminu) / naliczono karę (pominięte PK)  
**⚠️ Bardzo szczątkowy — brak reguł decyzyjnych, brak odwołań**

### Przepływ 10: Zakończenie zawodów
Zakończono wszystkie dystanse → organizator → przyznano nagrody  
Zawodnik → zwrócono pakiet → kaucja zwrócona  
Brak zwrotu w terminie → kaucja zatrzymana  
Zawodnik → otrzymano dyplom  
**⚠️ Brak procesu obliczania wyników/rankingu który byłby podstawą nagród**

---

## Znane reguły biznesowe (do weryfikacji w tej fazie)

1. W trakcie opłacania nie można zmienić dystansu
2. Zmiana grupy możliwa tylko w oknie czasowym
3. Zmiana grupy: jedna zmiana na zawodnika
4. Zmiana grupy: tylko na ten sam dystans
5. Generowanie grup: min. liczba uczestników z numerami
6. Generowanie grup: termin z regulaminu
7. Kaucja za dyskietkę — wysokość nieokreślona
8. Czas zwrotu pakietu startowego z regulaminu
9. Limity czasowe na dystansie z regulaminu
10. Promocja -100% — warunki nieokreślone

---

## Otwarte pytania (blocker) — priorytety dla tej fazy

1. **Q-001:** Jak obsługiwany jest konflikt generowania list vs niedopłata? → decyzja projektowa
2. **Q-003:** Czy Wyniki/Ranking to osobny BC? Jak jest obliczany i kiedy?
3. **Q-006:** Kto i kiedy przyznaje nagrody — czy to ręczna decyzja czy automatyczna?

---

## Wskazówki dla agenta Process Level

- Obszar **Nakładanie Kar** wymaga dopytania użytkownika — obecne dane niewystarczające
- Obszar **Wyniki/Ranking** nie istnieje w materiałach — wymaga jawnej decyzji
- Sprawdź zależności między BC (szczególnie Opłaty → Grupy Startowe → Start)
- Potwierdź czy **Konfiguracja Maratonu** to Generic BC (może być poddomieną Regulaminu)
