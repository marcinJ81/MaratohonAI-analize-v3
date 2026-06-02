---
phase: process-level
session: maraton-rowerowy
generated: 2026-06-02
status: completed
coverage: 78%
sources:
  - state/inputs/processed/phase-2-seed.md
  - state/inputs/processed/all-materials-processed.md
  - user-input (sesja 2026-06-02)
---

# Process Level — Maraton Rowerowy

## Kontekst

Analiza procesu obsługi maratonu rowerowego obejmująca pełen cykl życia uczestnika: od rejestracji przez start, pomiar czasu i ranking, do zakończenia zawodów. Wiedza pochodzi z 18 zdjęć Event Stormingu (60% pokrycia) uzupełnionych odpowiedziami eksperta domenowego (Q-001, Q-003, Q-004, Q-006). Obszary Nakładania Kar i Zakończenia mają niższe pokrycie — szczegóły powinny być doprecyzowane przed Design Level.

---

## Core Domain

**Nazwa:** Pomiar i Wyniki

**Uzasadnienie:** Rzetelny pomiar czasu z dual-write (lokalny zapis na punkcie + sync do głównej bazy), live ranking aktualizowany z każdego pomiaru między-czasowego oraz możliwość weryfikacji obecności zawodnika na każdym punkcie kontrolnym — to jest unikalna wartość systemu. Bez tego komponentu system jest tylko formularzem rejestracyjnym. Lokalny zapis jest jedynym dowodem obecności zawodnika na punkcie i stanowi fundament rozstrzygania sporów.

---

## Bounded Contexts

| ID     | Nazwa                   | Odpowiedzialność                                                                 | Typ        |
|--------|-------------------------|---------------------------------------------------------------------------------|------------|
| BC-001 | Konfiguracja Maratonu   | Parametry zawodów, regulamin, terminy, limity czasowe, zasady promocji          | Generic    |
| BC-002 | Rejestracja             | Zapisy uczestników, wybór dystansu, zmiana dystansu, stan uczestnika            | Supporting |
| BC-003 | Opłaty                  | Płatności, niedopłaty, nadpłaty, promocje -100%, zwroty                         | Supporting |
| BC-004 | Grupy Startowe          | Generacja grup, przypisanie numerów startowych, zmiany grup                     | Supporting |
| BC-005 | Przygotowanie Startu    | Weryfikacja tożsamości, wydanie pakietów startowych, dyskietki, kaucja          | Supporting |
| BC-006 | Start                   | Zamknięcie zapisów, rejestracja czasu startu grup i indywidualnego              | Supporting |
| BC-007 | Pomiar i Wyniki         | Rejestracja przejazdów, podliczanie czasów, live ranking, weryfikacja trasy     | **Core**   |
| BC-008 | Kary i Dyskwalifikacje  | Nakładanie kar przez sędziego/organizatora, degradacja, odwołania               | Supporting |
| BC-009 | Zakończenie i Nagrody   | Dyplomy, zwrot pakietów startowych, kaucja, nagrody za czas (z rankingu)        | Supporting |

---

## Przepływy procesów

### Rejestracja uczestnika — BC-002

**Wyzwalacz:** Uczestnik wypełnia formularz zapisu
**Rezultat:** Uczestnik zarejestrowany, przypisany dystans, wyliczona opłata, oczekiwanie na płatność

**Kroki:**
1. Uczestnik podaje dane i wybiera dystans
2. System waliduje dane i termin zapisów
3. System wylicza kwotę opłaty (na podstawie BC-001)
4. Zarejestrowano uczestnika — status: oczekuje na opłatę
5. Przekazano informację o należnej kwocie do BC-003

**Reguły biznesowe:**
| ID     | Reguła                                                         | Egzekwowana w kroku |
|--------|----------------------------------------------------------------|---------------------|
| RB-001 | Zapisy tylko w terminie określonym w regulaminie               | 2                   |
| RB-002 | W trakcie trwającego procesu opłacania zmiana dystansu jest zablokowana | 1          |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                          | Akcja                              | Skutek                             |
|--------|----------------------------------|------------------------------------|------------------------------------|
| EX-001 | Termin zapisów minął             | Odmowa rejestracji                 | Uczestnik nie zostaje zapisany     |
| EX-002 | Uczestnik chce zmienić dystans   | Wyliczono różnicę opłaty → BC-003  | Zmiana pending do czasu opłaty     |

---

### Zmiana dystansu — BC-002 + BC-003

**Wyzwalacz:** Uczestnik składa wniosek o zmianę dystansu
**Rezultat:** Dystans zmieniony i opłacony LUB wniosek odrzucony

**Kroki:**
1. Uczestnik zgłasza chęć zmiany dystansu
2. System sprawdza czy nie trwa aktywny proces opłacania
3. System wylicza różnicę opłaty (nowy dystans vs stary)
4. Jeśli niedopłata → stan: oczekuje na dopłatę (dystans NIE jest zmieniany)
5. Po zaksięgowaniu dopłaty → zmieniono dystans, zaktualizowano stan uczestnika

**Reguły biznesowe:**
| ID     | Reguła                                                                              | Egzekwowana w kroku |
|--------|-------------------------------------------------------------------------------------|---------------------|
| RB-003 | Zmiana dystansu finalizuje się dopiero po opłaceniu — uczestnik pozostaje na starym dystansie do momentu zaksięgowania płatności | 4–5 |
| RB-002 | W trakcie trwającego procesu opłacania zmiana dystansu jest zablokowana             | 2                   |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                                     | Akcja                           | Skutek                                              |
|--------|---------------------------------------------|---------------------------------|-----------------------------------------------------|
| EX-003 | Nowy dystans tańszy → nadpłata              | BC-003 oblicza zwrot            | Dystans zmieniony, zwrot inicjowany                 |
| EX-004 | Uczestnik nie opłaca niedopłaty             | Uczestnik pozostaje na starym dystansie i liście startowej | Brak dostępu do nowego dystansu |

---

### Opłata za udział — BC-003

**Wyzwalacz:** Uczestnik dokonuje przelewu bankowego
**Rezultat:** Opłacono uczestnictwo, uczestnik aktywny

**Kroki:**
1. Zarejestrowano przelew bankowy
2. System dopasowuje przelew do uczestnika
3. Sprawdzono czy kwota pokrywa należność
4. Opłacono uczestnictwo — zaktualizowano stan uczestnika w BC-002

**Reguły biznesowe:**
| ID     | Reguła                                                              | Egzekwowana w kroku |
|--------|---------------------------------------------------------------------|---------------------|
| RB-004 | Promocja -100% może być przyznana przez obsługę PK (warunki w regulaminie BC-001) | 3 |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek              | Akcja                                        | Skutek                              |
|--------|----------------------|----------------------------------------------|-------------------------------------|
| EX-005 | Kwota za niska       | Status: niedopłata, uczestnik informowany    | Uczestnik nie jest aktywowany       |
| EX-006 | Kwota za wysoka      | Wyliczono kwotę zwrotu, inicjowano zwrot     | Uczestnik aktywowany, zwrot wysłany |

---

### Generacja grup startowych — BC-004

**Wyzwalacz:** Nadejście terminu z regulaminu LUB ręczna decyzja administratora
**Rezultat:** Grupy startowe wygenerowane, numery startowe przypisane

**Kroki:**
1. System sprawdza czy minął termin z regulaminu (BC-001)
2. System sprawdza minimalną liczbę uczestników z przypisanymi numerami
3. Automatyczne przypisanie uczestników do grup według dystansu
4. Przypisano numery startowe
5. Wygenerowano grupy — zaktualizowano stan uczestników

**Reguły biznesowe:**
| ID     | Reguła                                                               | Egzekwowana w kroku |
|--------|----------------------------------------------------------------------|---------------------|
| RB-005 | Generowanie tylko gdy osiągnięto minimalną liczbę uczestników z numerami | 2               |
| RB-006 | Termin generowania grup z regulaminu (BC-001)                        | 1                   |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                                           | Akcja                                     | Skutek                                         |
|--------|---------------------------------------------------|-------------------------------------------|------------------------------------------------|
| EX-007 | Uczestnik opłaca dystans PO wygenerowaniu grup    | Przyznano numer, dołączono do istniejącej grupy | Uczestnik w grupie, może startować         |
| EX-008 | Administrator generuje grupy ręcznie              | Pomija sprawdzenie terminu                | Grupy wygenerowane niezależnie od harmonogramu |

---

### Zmiana grupy startowej — BC-004

**Wyzwalacz:** Uczestnik składa wniosek o zmianę grupy
**Rezultat:** Zmiana zaakceptowana lub odrzucona

**Kroki:**
1. Uczestnik zgłasza żądanie zmiany grupy
2. System sprawdza wszystkie guardy (patrz RB)
3. Jeśli wszystkie warunki spełnione → zmieniono grupę
4. Jeśli warunek niespełniony → poinformowano uczestnika o przyczynie

**Reguły biznesowe:**
| ID     | Reguła                                                        | Egzekwowana w kroku |
|--------|---------------------------------------------------------------|---------------------|
| RB-007 | Zmiana grupy możliwa tylko w oknie czasowym z regulaminu      | 2                   |
| RB-008 | Każdy uczestnik może zmienić grupę tylko raz                  | 2                   |
| RB-009 | Zmiana grupy tylko na grupę tego samego dystansu              | 2                   |
| RB-010 | Docelowa grupa musi mieć wolne miejsca                        | 2                   |
| RB-011 | Grupy muszą być już wygenerowane                              | 2                   |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                          | Akcja               | Skutek                    |
|--------|----------------------------------|---------------------|---------------------------|
| EX-009 | Termin zmiany grup minął         | Odmowa              | Uczestnik pozostaje w grupie |
| EX-010 | Brak wolnych miejsc w docelowej  | Odmowa              | Uczestnik pozostaje w grupie |

---

### Przygotowanie do startu — BC-005

**Wyzwalacz:** Uczestnik zgłasza się do obsługi PK przed zawodami
**Rezultat:** Uczestnik zweryfikowany i wyposażony LUB zdyskwalifikowany

**Kroki:**
1. Obsługa PK weryfikuje tożsamość uczestnika
2. Uczestnik podpisuje oświadczenie
3. Wydano pakiet startowy i dyskietkę
4. Pobrano kaucję za dyskietkę
5. Uczestnik gotowy do startu

**Reguły biznesowe:**
| ID     | Reguła                                           | Egzekwowana w kroku |
|--------|--------------------------------------------------|---------------------|
| RB-012 | Kaucja pobierana za każdą dyskietkę              | 4                   |
| RB-013 | Weryfikacja tożsamości obowiązkowa               | 1                   |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                          | Akcja                                 | Skutek                                          |
|--------|----------------------------------|---------------------------------------|-------------------------------------------------|
| EX-011 | Uczestnik nie przeszedł weryfikacji | Dyskwalifikacja                     | Brak prawa do startu                            |
| EX-012 | Zgubiona dyskietka               | Pobrano kaucję, wydano nową dyskietkę | Uczestnik może startować z nową dyskietką       |

---

### Start zawodników — BC-006

**Wyzwalacz:** Decyzja administratora o starcie zawodów
**Rezultat:** Grupy wystartowane, czasy startu zarejestrowane, zapisy zamknięte

**Kroki:**
1. Administrator zamyka zapisy na zawody
2. Weryfikacja listy startowej (porównanie z BC-005)
3. Sędzia trasy daje sygnał startu dla grupy
4. Zarejestrowano czas startu grupy → przekazano do BC-007
5. Wariant: start indywidualny — wyciągnięcie uczestnika z grupy

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                               | Akcja                             | Skutek                               |
|--------|---------------------------------------|-----------------------------------|--------------------------------------|
| EX-013 | Uczestnik nie na liście startowej     | Dyskwalifikacja przed startem     | Brak prawa do startu                 |
| EX-014 | Start indywidualny                    | Wyciągnięcie z grupy, osobny czas | Uczestnik startuje poza grupą        |
| EX-015 | Sędzia zdyskwalifikuje przed startem  | Odnotowano dyskwalifikację        | Uczestnik nie startuje               |

---

### Pomiar czasu i ranking live — BC-007

**Wyzwalacz:** Uczestnik mija punkt pomiarowy (dyskietka odczytana)
**Rezultat:** Czas zarejestrowany lokalnie i globalnie, ranking zaktualizowany

**Kroki:**
1. Dyskietka odczytana na punkcie pomiarowym
2. **Zapis lokalny** (source of truth dla weryfikacji) — odczyt zapisany na urządzeniu punktu
3. Sync do głównej bazy (jeśli połączenie dostępne; kolejkowanie gdy brak połączenia)
4. System oblicza czas między-odcinkowy
5. Zaktualizowano ranking live dla dystansu
6. Wykryto anomalię czasową → flag do BC-008 (jeśli czasy nie sumują się)

**Reguły biznesowe:**
| ID     | Reguła                                                                                        | Egzekwowana w kroku |
|--------|-----------------------------------------------------------------------------------------------|---------------------|
| RB-014 | Każdy odczyt zapisywany lokalnie niezależnie od połączenia z główną bazą                      | 2                   |
| RB-015 | Lokalny zapis jest jedynym dowodem obecności zawodnika na punkcie                             | 2                   |
| RB-016 | Ranking obliczany na podstawie czasu startu (BC-006) + czasy odczytów + czas końcowy          | 5                   |
| RB-017 | Limity czasowe na dystansie z regulaminu (BC-001) — po przekroczeniu zakończenie przedwczesne | 6                   |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                                         | Akcja                                          | Skutek                                              |
|--------|-------------------------------------------------|------------------------------------------------|-----------------------------------------------------|
| EX-016 | Brak połączenia z główną bazą                   | Zapis lokalny, sync po przywróceniu połączenia | Dane nie są utracone                                |
| EX-017 | Uczestnik chce jechać dalej po limicie          | Obsługa PK → przedłużono przejazd, dodatkowy pomiar | Odnotowano jako przedłużenie                   |
| EX-018 | Anomalia czasowa (skrócenie trasy)              | Flaga do BC-008 do weryfikacji przez sędziego  | Możliwa dyskwalifikacja                             |
| EX-019 | Wszyscy dojechali LUB minął czas → koniec dystansu | Zakończono dystans → trigger do BC-009     | Dystans zakończony, wyniki finalne                  |

---

### Nakładanie kar i dyskwalifikacje — BC-008

**Wyzwalacz:** Decyzja sędziego trasy lub organizatora (ręczna)
**Rezultat:** Kara nałożona, uczestnik zdyskwalifikowany lub zdegradowany na koniec listy

**Kroki:**
1. Sędzia trasy lub organizator stwierdza naruszenie regulaminu
2. Weryfikacja (opcjonalnie: sprawdzenie danych z BC-007)
3. Nałożono karę: dyskwalifikacja LUB degradacja na koniec listy
4. Zaktualizowano wynik uczestnika w BC-007

**Reguły biznesowe:**
| ID     | Reguła                                                                                | Egzekwowana w kroku |
|--------|---------------------------------------------------------------------------------------|---------------------|
| RB-018 | Kary nakładane wyłącznie przez sędziego trasy lub organizatora — brak automatycznych kar | 3                |
| RB-019 | Skrócenie trasy (niespójne czasy między punktami) → dyskwalifikacja lub koniec listy  | 2–3                 |
| RB-020 | Brak doliczania czasu — kara to dyskwalifikacja albo degradacja na koniec listy       | 3                   |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                          | Akcja                                       | Skutek                               |
|--------|----------------------------------|---------------------------------------------|--------------------------------------|
| EX-020 | Zawodnik kwestionuje karę/wynik  | Odwołanie do sędziego trasy lub organizatora | Decyzja po weryfikacji danych z BC-007 |

---

### Zakończenie zawodów i nagrody — BC-009

**Wyzwalacz:** Zakończenie wszystkich dystansów (trigger z BC-007)
**Rezultat:** Nagrody przyznane, dyplomy wydane, pakiety zwrócone, kaucje rozliczone

**Kroki:**
1. Odebrano sygnał zakończenia wszystkich dystansów
2. System generuje dyplomy dla uczestników którzy ukończyli dystans
3. Nagrody za najlepszy czas per dystans — wynikają bezpośrednio z rankingu BC-007
4. Nagrody specjalne organizatora — poza systemem
5. Uczestnik zwraca pakiet startowy
6. Zwrócono kaucję za dyskietkę (jeśli pakiet zwrócony w terminie)

**Reguły biznesowe:**
| ID     | Reguła                                                               | Egzekwowana w kroku |
|--------|----------------------------------------------------------------------|---------------------|
| RB-021 | Każdy uczestnik który ukończył dystans otrzymuje dyplom i nagrodę za przejazd | 2          |
| RB-022 | Nagrody za najlepszy czas wynikają z rankingu BC-007 — nie wymagają ręcznej decyzji | 3   |
| RB-023 | Kaucja zatrzymana jeśli pakiet nie zwrócony w terminie               | 6                   |
| RB-024 | Nagrody specjalne organizatora są przyznawane poza systemem          | 4                   |

**Wyjątki i ścieżki alternatywne:**
| ID     | Warunek                                   | Akcja                    | Skutek                        |
|--------|-------------------------------------------|--------------------------|-------------------------------|
| EX-021 | Brak zwrotu pakietu w terminie            | Kaucja zatrzymana        | Uczestnik traci kaucję        |
| EX-022 | Uczestnik nie ukończył dystansu           | Brak dyplomu i nagrody   | Tylko uczestnictwo odnotowane |

---

## Zależności między kontekstami

| ID    | Z (BC)                  | Do (BC)                 | Typ relacji          | Kierunek         | Uwagi                                                        |
|-------|-------------------------|-------------------------|----------------------|------------------|--------------------------------------------------------------|
| D-001 | BC-001 Konfiguracja     | BC-002 Rejestracja      | upstream-downstream  | synchroniczny    | Terminy, reguły zapisów                                      |
| D-002 | BC-001 Konfiguracja     | BC-003 Opłaty           | upstream-downstream  | synchroniczny    | Stawki, promocje, warunki                                    |
| D-003 | BC-001 Konfiguracja     | BC-004 Grupy Startowe   | upstream-downstream  | synchroniczny    | Termin generowania, minimalna liczba uczestników             |
| D-004 | BC-001 Konfiguracja     | BC-007 Pomiar i Wyniki  | upstream-downstream  | synchroniczny    | Limity czasowe na dystansach                                 |
| D-005 | BC-002 Rejestracja      | BC-003 Opłaty           | upstream-downstream  | synchroniczny    | Rejestracja inicjuje płatność, zmiana dystansu → nowa kwota  |
| D-006 | BC-003 Opłaty           | BC-002 Rejestracja      | upstream-downstream  | asynchroniczny   | Potwierdzenie płatności → finalizacja zmiany dystansu        |
| D-007 | BC-003 Opłaty           | BC-004 Grupy Startowe   | upstream-downstream  | asynchroniczny   | Opłacenie → uprawnienie do numeru startowego                 |
| D-008 | BC-004 Grupy Startowe   | BC-005 Przygot. Startu  | upstream-downstream  | synchroniczny    | Lista startowa → weryfikacja uczestnika                      |
| D-009 | BC-005 Przygot. Startu  | BC-006 Start            | upstream-downstream  | synchroniczny    | Zweryfikowani uczestnicy → lista do startu                   |
| D-010 | BC-006 Start            | BC-007 Pomiar i Wyniki  | upstream-downstream  | synchroniczny    | Czas startu grupy → punkt odniesienia dla rankingu           |
| D-011 | BC-007 Pomiar i Wyniki  | BC-008 Kary             | upstream-downstream  | asynchroniczny   | Anomalie czasowe flagowane do weryfikacji przez sędziego     |
| D-012 | BC-008 Kary             | BC-007 Pomiar i Wyniki  | upstream-downstream  | synchroniczny    | Decyzja o karze → aktualizacja wyniku/rankingu               |
| D-013 | BC-007 Pomiar i Wyniki  | BC-009 Zakończenie      | upstream-downstream  | asynchroniczny   | Zakończenie dystansu → trigger zakończenia, dane rankingu    |

---

## Polityki (Policies)

| ID    | Zdarzenie wyzwalające              | Polityka / Reguła                                                   | Akcja / Komenda                         | BC       |
|-------|------------------------------------|---------------------------------------------------------------------|-----------------------------------------|----------|
| P-001 | Zmieniono dystans uczestnika       | Gdy zmiana dystansu, oblicz różnicę opłaty i zablokuj finalizację do opłaty | Inicjuj proces niedopłaty/nadpłaty | BC-002/003 |
| P-002 | Zaksięgowano dopłatę               | Gdy dopłata pokrywa niedopłatę, finalizuj zmianę dystansu           | Aktywuj nowy dystans dla uczestnika     | BC-003→002 |
| P-003 | Nadszedł termin generowania grup   | Gdy termin z regulaminu i min. liczba uczestników → generuj grupy   | Uruchom generację grup                  | BC-001→004 |
| P-004 | Odczytano dyskietkę na punkcie     | Zapisz lokalnie zawsze, sync do bazy gdy połączenie                 | Dual-write + aktualizacja rankingu      | BC-007   |
| P-005 | Anomalia czasowa wykryta           | Gdy czasy między punktami nie sumują się → przekaż do weryfikacji   | Flaga do sędziego trasy                 | BC-007→008 |
| P-006 | Zakończono ostatni dystans         | Gdy wszystkie dystanse zakończone → inicjuj zakończenie zawodów     | Trigger zakończenia + generacja dyplomów | BC-007→009 |
| P-007 | Uczestnik ukończył dystans         | Każdy kto przekroczy meta otrzymuje dyplom i nagrodę za przejazd    | Generuj dyplom, odznacz nagrodę         | BC-009   |
| P-008 | Upłynął termin zwrotu pakietu      | Gdy pakiet nie zwrócony w terminie → zatrzymaj kaucję               | Odnotuj brak zwrotu, zatrzymaj kaucję   | BC-009   |

---

## Wnioski

| ID    | Wniosek                                                                                                        | Pewność  | Źródło               |
|-------|----------------------------------------------------------------------------------------------------------------|----------|----------------------|
| W-001 | Core Domain to Pomiar i Wyniki — dual-write i live ranking są unikalną wartością systemu                       | wysoka   | user-input           |
| W-002 | Zmiana dystansu jest procesem dwufazowym: wniosek → pending, finalizacja po opłacie                           | wysoka   | user-input (Q-001)   |
| W-003 | Lokalny zapis na punkcie pomiarowym jest jedynym i wyłącznym dowodem obecności zawodnika                      | wysoka   | user-input (Q-003)   |
| W-004 | Ranking jest obliczany live z każdego pomiaru między-czasowego — nie jest batch po zawodach                   | wysoka   | user-input (Q-003)   |
| W-005 | Kary to dyskwalifikacja lub degradacja na koniec listy — brak doliczania czasu                                | wysoka   | user-input (Q-004)   |
| W-006 | Nakładanie kar jest wyłącznie ręczne (sędzia/organizator) — brak automatycznych kar                           | wysoka   | user-input (Q-004)   |
| W-007 | System odpowiada za nagrody za czas (z rankingu), nagrody specjalne organizatora są poza systemem             | wysoka   | user-input (Q-006)   |
| W-008 | BC-001 Konfiguracja Maratonu jest Generic — może być zastąpiony przez zewnętrzny CMS lub prosty config       | średnia  | derived              |
| W-009 | Obszar odwołań od kar nie ma zdefiniowanego procesu — decyzja ludzka ad hoc                                   | wysoka   | user-input (Q-004)   |
| W-010 | BC-003 Opłaty może być kandydatem do zastąpienia zewnętrznym systemem płatniczym (anti-corruption layer)      | średnia  | derived              |

---

## Otwarte pytania

| ID    | Pytanie                                                                                          | Priorytet      | Dotyczy          | Przeniesione do fazy |
|-------|--------------------------------------------------------------------------------------------------|----------------|------------------|-----------------------|
| Q-007 | Jak wygląda sync lokalny→globalny gdy punkt był offline dłuższy czas — kolejkowanie czy merge?  | blocker        | BC-007           | design-level          |
| Q-008 | Czy zawodnik widzi swój live ranking podczas zawodów, czy tylko na mecie?                       | nice-to-have   | BC-007           | design-level          |
| Q-009 | Jakie dokładnie dane trafiają do dyplomu — czy jest generowany automatycznie czy ręcznie?       | nice-to-have   | BC-009           | design-level          |
| Q-010 | Czy obsługa odwołań (zakwestionowanie wyniku/kary) wymaga śledzonego procesu w systemie?        | blocker        | BC-008           | design-level          |
| Q-011 | Warunki promocji -100% — kto i kiedy może ją przyznać, czy są limity?                          | nice-to-have   | BC-003           | design-level          |
| Q-012 | Czy istnieje pojęcie "kategorii" zawodnika (wiek, płeć) wpływające na ranking lub nagrody?     | blocker        | BC-007, BC-009   | design-level          |

---

## Korekty i uzupełnienia (retrospektywne)

<!-- Sekcja wypełniana przez orkiestratora gdy późniejsza faza zmienia wnioski tej fazy. -->
