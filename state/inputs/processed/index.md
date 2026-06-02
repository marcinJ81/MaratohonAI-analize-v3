# Inputs Index — Maraton Rowerowy

| Plik | Typ | Konwersja | Użyty w fazach | Pokrycie BP | Pokrycie PL | Pokrycie DL |
|------|-----|-----------|----------------|-------------|-------------|-------------|
| ogólny_widok_kontekstu_rejestracja.png | obraz ES | lossy | phase-1, phase-2 | wysoki | niski | brak |
| kontekst_rejestracja1.png | obraz ES | lossy | phase-1, phase-2 | wysoki | średni | brak |
| opłata_za_udział_1.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| opłata_za_udział_2.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| generacja grup startowych.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| generacja_grup_poterminie.png | obraz ES | lossy | phase-1, phase-2 | średni | wysoki | brak |
| grupy_startowe_ds.png | obraz DS | lossy | phase-3 | brak | brak | wysoki |
| genreacja_numeru_indywidualna.png | obraz DS | lossy | phase-2, phase-3 | brak | średni | wysoki |
| kary.png | obraz ES | lossy | phase-1, phase-2 | niski | niski | brak |
| pomiar_czasu.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| przypisanie_zawodnika_do_listy.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| start_indywidualny.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| utowrzenie_grup_ds.png | obraz DS | lossy | phase-3 | brak | brak | wysoki |
| wystartowano_zawodników.png | obraz ES | lossy | phase-1, phase-2 | wysoki | średni | brak |
| zakończenie_2.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| zakończenie_zawodów.png | obraz ES | lossy | phase-1, phase-2 | wysoki | wysoki | brak |
| zmiania_grupy_startowej_DL.png | obraz DS+ES | lossy | phase-2, phase-3 | średni | wysoki | wysoki |
| zminia_grupystartowej.png | obraz DS+ES | lossy | phase-2, phase-3 | średni | wysoki | wysoki |

## Ocena pokrycia per faza

| Faza | Pokrycie | Uzasadnienie |
|------|----------|--------------|
| Big Picture | **~80%** | Aktorzy, zdarzenia, granice dobrze widoczne. Brak: wyniki/ranking, anulowanie rejestracji. |
| Process Level | **~60%** | Przepływy główne obecne. Braki: kary (szczątkowe), wyniki/ranking (brak), reguły decyzyjne w kilku miejscach niejasne. Hot spot HS-01 nierozwiązany. |
| Design Level | **~15%** | Tylko 2 agregaty (GrupyStartowe). Brak: Rejestracja, Płatność, Zawodnik, Pomiar Czasu, Nagrody. |
