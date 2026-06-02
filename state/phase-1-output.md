---
phase: big-picture
session: maraton-rowerowy
generated: 2026-06-01
status: completed
coverage: 80%
sources:
  - state/inputs/processed/all-materials-processed.md
  - Dane_wejsciowe/ (18 obrazów PNG)
---

## Kontekst

Analizowany system obsługuje maraton rowerowy — od konfiguracji zawodów, przez rejestrację i opłaty, generację grup startowych, start, pomiar czasu, aż do zakończenia i wręczenia nagród. System ma 8 wyróżnionych obszarów domenowych i integruje się z zewnętrznym systemem bankowym. Złożoność: **średnia-wysoka**.

---

## Aktorzy

| ID   | Nazwa                       | Rola                                                    | Zewnętrzny? |
|------|-----------------------------|----------------------------------------------------------|-------------|
| A-01 | Uczestnik / Zawodnik        | Rejestruje się, opłaca, startuje, jedzie trasę           | Nie         |
| A-02 | Administrator               | Konfiguruje maraton, zarządza listami, obsługuje wyjątki | Nie         |
| A-03 | Główny Organizator Zawodów  | Zatwierdza płatności, przyznaje nagrody                  | Nie         |
| A-04 | Sędzia Trasy                | Startuje grupy, rejestruje czasy startu, nakłada kary    | Nie         |
| A-05 | Obsługa Punktu Kontrolnego  | Weryfikuje zawodników, wydaje pakiety, obsługuje kaucje  | Nie         |
| A-06 | Pracownik Obsługi           | Obsługuje zwroty nadpłat                                 | Nie         |
| A-07 | Interfejs Banku             | Przetwarza przelewy                                      | Tak         |
| A-08 | System Pomiaru Czasu        | Rejestruje przyłożenia dyskietki, podlicza czasy         | Nie (system)|

---

## Zdarzenia domenowe

| ID    | Zdarzenie                                 | Aktor/Trigger           | Obszar                    |
|-------|-------------------------------------------|--------------------------|---------------------------|
| E-001 | Zarejestrowano maraton                    | Administrator            | Konfiguracja Maratonu     |
| E-002 | Zarejestrowano uczestnika                 | Uczestnik                | Rejestracja               |
| E-003 | Wybrano dystans                           | Uczestnik                | Rejestracja               |
| E-004 | Potwierdzono dane kontaktowe              | Uczestnik                | Rejestracja               |
| E-005 | Zweryfikowano zawodnika                   | System/obsługa           | Rejestracja               |
| E-006 | Złożono wniosek o zmianę dystansu         | Uczestnik                | Rejestracja               |
| E-007 | Zmieniono dystans                         | System                   | Rejestracja               |
| E-008 | Wyliczono opłatę                          | System                   | Rejestracja               |
| E-009 | Poinformowano użytkownika o zmianie opłaty| System                   | Rejestracja               |
| E-010 | Rozpoczęto opłacenie udziału              | Uczestnik                | Opłata za Udział          |
| E-011 | Zarejestrowano przelew                    | Interfejs Banku          | Opłata za Udział          |
| E-012 | Zarejestrowano opłatę                     | Obsługa PK               | Opłata za Udział          |
| E-013 | Zaaplikowano promocje (-100%)             | System                   | Opłata za Udział          |
| E-014 | Opłacono uczestnictwo                     | System                   | Opłata za Udział          |
| E-015 | Rozpoczęto opłacenie niedopłaty           | Uczestnik                | Opłata za Udział          |
| E-016 | Opłacono niedopłatę                       | System                   | Opłata za Udział          |
| E-017 | Opłacono zmianę dystansu                  | System                   | Opłata za Udział          |
| E-018 | Wyliczono kwotę zwrotu                    | System                   | Opłata za Udział          |
| E-019 | Zwrócono nadpłatę                         | Pracownik Obsługi        | Opłata za Udział          |
| E-020 | Przypisano zawodnika do grupy             | Administrator/System     | Generacja Grup            |
| E-021 | Przypisano numer startowy                 | System                   | Generacja Grup            |
| E-022 | Złożono wniosek o zmianę grupy            | Zawodnik/Administrator   | Generacja Grup            |
| E-023 | Zmieniono grupę                           | System                   | Generacja Grup            |
| E-024 | Nie udało się zmienić grupy               | System                   | Generacja Grup            |
| E-025 | Wygenerowano grupy startowe               | System/Administrator     | Generacja Grup            |
| E-026 | Nie udało się wygenerować grup startowych | System                   | Generacja Grup            |
| E-027 | Dołączono do grupy po terminie            | System                   | Generacja Grup            |
| E-028 | Zweryfikowano tożsamość zawodnika         | Obsługa PK               | Przygotowanie do Startu   |
| E-029 | Zawodnik nie przeszedł weryfikacji        | Obsługa PK               | Przygotowanie do Startu   |
| E-030 | Podpisano oświadczenie zawodnika          | Zawodnik                 | Przygotowanie do Startu   |
| E-031 | Wydano pakiet startowy                    | Obsługa PK               | Przygotowanie do Startu   |
| E-032 | Zdyskwalifikowano zawodnika               | Obsługa PK / Sędzia      | Przygotowanie do Startu   |
| E-033 | Zweryfikowano zawodnika przed startem     | Obsługa PK               | Przygotowanie do Startu   |
| E-034 | Zgłoszono indywidualny start              | Zawodnik                 | Przygotowanie do Startu   |
| E-035 | Wyciągnięto zawodnika z grupy             | Administrator            | Przygotowanie do Startu   |
| E-036 | Zgubiono dyskietkę z numerem startowym    | Zawodnik                 | Przygotowanie do Startu   |
| E-037 | Wydano nową dyskietkę                     | Obsługa PK               | Przygotowanie do Startu   |
| E-038 | Zamknięto zapisy na dany dystans          | Administrator            | Start                     |
| E-039 | Wystartowano grupę                        | Sędzia Trasy             | Start                     |
| E-040 | Zarejestrowano czas startu                | Sędzia Trasy             | Start                     |
| E-041 | Wystartowano zawodnika (indywidualnie)    | Sędzia Trasy             | Start                     |
| E-042 | Zakończono udział przedwcześnie           | Sędzia Trasy             | Start                     |
| E-043 | Zarejestrowano pomiar czasu               | System Pomiaru Czasu     | Pomiar Czasu              |
| E-044 | Zawodnik zakończył przejazd               | System Pomiaru Czasu     | Pomiar Czasu              |
| E-045 | Podliczono czasy przejazdu                | System Pomiaru Czasu     | Pomiar Czasu              |
| E-046 | Zakończono przejazd zawodników na dystansie| System                  | Pomiar Czasu              |
| E-047 | Zakończono przejazdy wszystkich dystansów | System                   | Pomiar Czasu              |
| E-048 | Zgłoszono prośbę o przedłużenie dystansu  | Zawodnik                 | Pomiar Czasu              |
| E-049 | Przedłużono przejazd zawodnika            | Obsługa PK               | Pomiar Czasu              |
| E-050 | Nałożono karę za złamanie regulaminu      | Sędzia Trasy             | Nakładanie Kar            |
| E-051 | Naliczono karę za pominięte PK            | System/Sędzia            | Nakładanie Kar            |
| E-052 | Przyznano nagrodę                         | Główny Organizator       | Zakończenie Zawodów       |
| E-053 | Zwrócono pakiet startowy                  | Zawodnik/uprawniony      | Zakończenie Zawodów       |
| E-054 | Kaucja została zwrócona                   | Obsługa PK               | Zakończenie Zawodów       |
| E-055 | Kaucja została zatrzymana                 | System/czas              | Zakończenie Zawodów       |
| E-056 | Otrzymano dyplom                          | Zawodnik                 | Zakończenie Zawodów       |

---

## Granice systemu i integracje zewnętrzne

| System/Integracja | Kierunek       | Opis                                                    |
|-------------------|----------------|---------------------------------------------------------|
| Interfejs Banku   | Dwukierunkowy  | Inicjacja przelewu przez uczestnika → potwierdzenie wpływu do systemu |

Notyfikacje do uczestnika sugerowane przez zdarzenie "poinformowano użytkownika" — mechanizm nieokreślony w materiałach.

---

## Hot Spoty

| ID    | Opis                                                                                  | Powód                                         | Powiązane zdarzenia        |
|-------|---------------------------------------------------------------------------------------|-----------------------------------------------|---------------------------|
| HS-01 | Co zrobić gdy rozpocznie się generowanie list a czekamy na opłacenie niedopłaty?      | Konflikt stanów między Opłatą a Generacją Grup | E-016, E-025              |
| HS-02 | Nakładanie kar — proces decyzyjny całkowicie nieudokumentowany                        | Brak przepływu, reguł odwołań                  | E-050, E-051              |
| HS-03 | Wyniki / Ranking — brak tego obszaru w materiałach                                    | Pominięto w ES?                                | E-045, E-047              |
| HS-04 | Anulowanie rejestracji — brak procesu                                                 | Pominięto w ES?                                | E-002                     |
| HS-05 | Notyfikacje — mechanizm nieokreślony                                                  | Cross-cutting concern                          | E-009                     |
| HS-06 | Kto i na jakiej podstawie przyznaje nagrody (ranking)?                                | Brak powiązania z wynikami                     | E-052                     |

---

## Wnioski

| ID    | Wniosek                                                                                       | Pewność  | Źródło      |
|-------|-----------------------------------------------------------------------------------------------|----------|-------------|
| W-001 | System ma 8 wyraźnych obszarów domenowych                                                     | wysoka   | ES images   |
| W-002 | Core domain to zarządzanie startem i pomiarem czasu — to odróżnia ten system od e-commerce    | średnia  | derived     |
| W-003 | Rejestracja i Opłata to osobne konteksty — zdarzenia i reguły są logicznie rozdzielne         | wysoka   | ES images   |
| W-004 | Generacja Grup jest złożona: ma warianty czasowe i regułowe — kandydat na Core lub Supporting | wysoka   | ES images   |
| W-005 | Interfejs Banku jest jedyną widoczną integracją zewnętrzną                                    | wysoka   | ES images   |
| W-006 | Brakuje całego obszaru Wyniki/Ranking — to może być osobny BC lub część Pomiaru Czasu         | wysoka   | brak w mat. |

## Otwarte pytania

| ID    | Pytanie                                                               | Priorytet    | Dotyczy             |
|-------|-----------------------------------------------------------------------|--------------|---------------------|
| Q-001 | Jak obsługiwany jest konflikt: generowanie list vs niedopłata?         | blocker      | Opłata ↔ Gen. Grup  |
| Q-002 | Czy istnieje proces anulowania rejestracji / rezygnacji?              | nice-to-have | Rejestracja         |
| Q-003 | Jak działa kontekst Wyniki/Ranking — czy jest w systemie?             | blocker      | brak w materiałach  |
| Q-004 | Kto i kiedy weryfikuje kary — czy zawodnik może się odwołać?          | nice-to-have | Nakładanie Kar      |
| Q-005 | Jaki mechanizm notyfikacji (email, SMS, push)?                        | nice-to-have | cross-cutting       |

## Korekty i uzupełnienia (retrospektywne)
<!-- wypełnia orkiestrator gdy późniejsza faza zmienia ten output -->
