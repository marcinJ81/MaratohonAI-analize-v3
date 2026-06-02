<!-- source: images (event-storming), session: maraton-rowerowy, phase: big-picture + process-level -->
<!-- used-in: phase-1, phase-2 -->
<!-- conversion: lossy — dane wyekstrahowane z 18 obrazów PNG -->

# Przetworzone materiały — Maraton Rowerowy

## 1. Aktorzy

| ID  | Nazwa                         | Rola domenowa                                              | Zewnętrzny? |
|-----|-------------------------------|------------------------------------------------------------|-------------|
| A-01 | Uczestnik / Zawodnik         | Rejestruje się, opłaca, startuje, jedzie trasę             | Nie         |
| A-02 | Administrator                | Konfiguruje maraton, zarządza listami, obsługuje wyjątki   | Nie         |
| A-03 | Główny Organizator Zawodów   | Zatwierdza płatności, przyznaje nagrody, decyduje o zwrotach | Nie       |
| A-04 | Sędzia Trasy                 | Startuje grupy, rejestruje czas startu, nakłada kary       | Nie         |
| A-05 | Obsługa Punktu Kontrolnego   | Weryfikuje zawodników, wydaje pakiety startowe, obsługuje kaucję | Nie   |
| A-06 | Pracownik Obsługi            | Obsługuje zwroty nadpłat                                   | Nie         |
| A-07 | Interfejs Banku              | Przetwarza przelewy (integracja zewnętrzna)                | Tak         |
| A-08 | System Pomiaru Czasu         | Rejestruje przyłożenia dyskietki, podlicza czasy           | Nie (system)|

---

## 2. Zdarzenia domenowe per obszar

### OBSZAR: Konfiguracja Maratonu
- Zarejestrowano maraton
- *(reguły konfiguracji maratonu — polityka)*

### OBSZAR: Rejestracja
- Zarejestrowano uczestnika
- Wybrano dystans
- Potwierdzono dane kontaktowe
- Zweryfikowano zawodnika
- Złożono wniosek o zmianę dystansu
- Zmieniono dystans
- Wyliczono opłatę
- Poinformowano użytkownika o zmianie opłaty

### OBSZAR: Opłata za Udział
- Rozpoczęto opłacenie udziału
- Zarejestrowano przelew *(via interfejs banku)*
- Zarejestrowano opłatę
- Zaaplikowano promocje (-100%)
- Opłacono uczestnictwo
- Rozpoczęto opłacenie niedopłaty
- Zarejestrowano przelew niedopłaty
- Opłacono niedopłatę
- Opłacono zmianę dystansu
- Wyliczono kwotę zwrotu
- Zwrócono nadpłatę

**Reguły biznesowe:**
- W trakcie opłacania nie można zmienić dystansu
- *Hot spot:* Co zrobić, gdy rozpocznie się generowanie list a czekamy na opłacenie niedopłaty?

### OBSZAR: Generacja Grup Startowych
- Przypisano zawodnika do grupy
- Przypisano numer startowy
- Złożono wniosek o zmianę grupy
- Przyjęto wniosek o zmianę grupy → Grupa została zmieniona
- Nie przyjęto wniosku o zmianę grupy → Grupa nie została zmieniona
- Zmieniono grupę (zawodnik)
- Nie udało się zmienić grupy
- Wygenerowano ostateczne grupy
- Wygenerowano grupy startowe (automatycznie)
- Nie udało się wygenerować grup startowych
- Dołączono do grupy po terminie

**Reguły biznesowe (DS — guardy):**
- Nie przekroczono ostatecznego terminu zmiany grup
- Grupa docelowa ma wolne miejsca
- Zawodnik nie zmieniał wcześniej grupy
- Grupa docelowa jest z tego samego dystansu
- Grupy startowe zostały wygenerowane
- Minimalna liczba uczestników spełniona
- Nie przekroczono terminu generowania grup
- Generowanie grup odbywa się w określonym w regulaminie terminie

**Warianty zmiany grupy:**
- Zmiana przed wygenerowaniem grup startowych
- Zmiana po wygenerowaniu grup startowych
- Zmiana po terminie (możliwa z ograniczeniami)

### OBSZAR: Przygotowanie do Startu
- Zweryfikowano tożsamość zawodnika (w biurze)
- Zawodnik nie przeszedł weryfikacji
- Podpisano oświadczenie zawodnika
- Zawodnik nie podpisał oświadczenia
- Wydano pakiet startowy
- Zdyskwalifikowano zawodnika
- Zweryfikowano zawodnika przed startem (na podstawie listy startowej)
- Zgłoszono indywidualny start
- Wyciągnięto zawodnika z grupy (start indywidualny, np. spóźnienie)
- Zgubiono dyskietkę z numerem startowym → przejęto kaucję → odpięto numer → przypisano numer → wydano nową dyskietkę

### OBSZAR: Zamknięcie Zapisów i Start
- Wystartowano zawodników na danym dystansie
- Zamknięto zapisy na dany dystans
- Wystartowano grupę
- Zarejestrowano czas startu
- Wystartowano zawodnika (indywidualnie)
- Zdyskwalifikowano zawodnika (przy starcie)
- Zakończono udział przedwcześnie

### OBSZAR: Pomiar Czasu
- Zarejestrowano pomiar czasu *(przyłożenie dyskietki do czytnika)*
- Zawodnik zakończył przejazd
- Podliczono czasy przejazdu
- Zakończono przejazd zawodników na danym dystansie
- Zakończono przejazdy wszystkich dystansów
- Zgłoszono prośbę o przedłużenie dystansu
- Przedłużono przejazd zawodnika
- Odnotowano dodatkowy pomiar czasu

**Reguły:**
- Limity czasowe na dany dystans (regulamin)
- System wie że wszyscy dojechali albo że minął czas

**Warianty:**
- Pomiar czasu zadeklarowanego dystansu (standardowy)
- Pomiar czasu na wydłużonym dystansie (zawodnik chce jechać dalej)

### OBSZAR: Nakładanie Kar
- Nałożono karę na zawodnika za złamanie regulaminu
- Naliczono karę za pominięte punkty kontrolne

**Uwaga:** Obszar bardzo skrótowo opisany — brak przepływu decyzji, brak informacji kto i kiedy weryfikuje kary, brak informacji o odwołaniach.

### OBSZAR: Zakończenie Zawodów
- Zakończono przejazdy wszystkich dystansów *(trigger)*
- Przyznano nagrodę
- Zwrócono pakiet startowy → Kaucja została zwrócona
- Nie zwrócono pakietu startowego w terminie → Kaucja została zatrzymana
- Otrzymano dyplom

---

## 3. Granice systemu i integracje zewnętrzne

| Integracja       | Kierunek         | Opis                                      |
|------------------|------------------|-------------------------------------------|
| Interfejs banku  | Dwukierunkowy    | Inicjacja przelewu → potwierdzenie wpływu |

Brak innych widocznych integracji zewnętrznych (email, SMS etc. nie są explicite nazwane, choć "poinformowano użytkownika" sugeruje notyfikacje).

---

## 4. Hot Spoty i pytania otwarte

| ID   | Opis                                                                                    | Priorytet | Obszar                              |
|------|-----------------------------------------------------------------------------------------|-----------|-------------------------------------|
| HS-01 | Co zrobić gdy rozpocznie się generowanie list a czekamy na opłacenie niedopłaty?       | blocker   | Opłata ↔ Generacja Grup             |
| HS-02 | Obszar Nakładania Kar jest bardzo skrótowy — brak przepływu decyzji, odwołań           | nice-to-have | Nakładanie Kar                   |
| HS-03 | Brak kontekstu Wyników / Rankingu — jak są obliczane, kto ma dostęp?                   | blocker   | Brak w materiałach                  |
| HS-04 | Anulowanie rejestracji — brak w materiałach                                            | nice-to-have | Rejestracja                      |
| HS-05 | Notyfikacje do uczestnika — "poinformowano użytkownika" bez szczegółów mechanizmu      | nice-to-have | Cross-cutting                    |
| HS-06 | Kto decyduje o przyznaniu nagrody i na jakiej podstawie?                               | blocker   | Zakończenie Zawodów                 |

---

## 5. Polityki / Reguły domenowe (różowe karteczki)

- Reguły konfiguracji maratonu (ogólna, z kontekstu Konfiguracja)
- W trakcie opłacania nie można zmienić dystansu
- Zmiana grupy jest możliwa tylko w pewnym oknie czasowym
- Regulamin maratonu (kary) — ogólne odwołanie
- Reguły zmiany grupy: typ limit, dystans
- Generowanie grup odbywa się w określonym w regulaminie terminie
- Limity czasowe na dystans (regulamin)
- Czas do zwrotu pakietu ustalony w regulaminie
- Promocja -100% — brak szczegółów kiedy i dla kogo

---

## 6. Wzorce Design Level (z diagramów DS)

Dwa agregaty widoczne:
- **GrupyStartowe** (grupy_startowe_ds.png + utowrzenie_grup_ds.png)
  - Command: Generuj / Zmień
  - Reads: konfiguracja (terminy), grupy startowe, historia zmian
- Brak DS dla pozostałych agreggatów (Rejestracja, Płatność, Zawodnik, etc.)
