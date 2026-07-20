# Plan dyżurów · STREFA FT / SUB

Aplikacja do układania harmonogramu sprzątania dla brygady. Jeden plik: `index.html`.

## Tryby

- **Tydzień** — tabela zadania × dni z przypisanymi osobami, przycisk Drukuj/PDF (układ poziomy A4).
- **Pracownik** — każdy klika w siebie i widzi: dziś, najbliższe, ostatnie dyżury (z kim, czy z wózkiem).
- **Brygadzista** (opcjonalny PIN w ustawieniach):
  - siatka **dyspozycyjności** na 7 dni (✓ / —), kropki statusu przy datach,
  - **Plany tygodnia** — układanie dnia jednym klikiem albo „Ułóż brakujące" na cały tydzień (wg zaznaczonej dyspo),
  - edytor planu: zmiana osób (przy nazwisku widać obłożenie w tygodniu), „Ktoś wypadł? → Zdejmij i zastąp" (odznacza też dyspo),
  - **Obłożenie** — paski na ten tydzień + suma z całej historii,
  - ustawienia: brygada (z uprawnieniami na wózki), zadania (waga, liczba osób, częstotliwość, stała osoba, „razem z"), dni robocze, PIN, reset.

Rotacją steruje balans: suma wag z historii + obciążenie danego dnia + świeżość
(kto robił zadanie niedawno, dostaje je później).

## Krok 1 — działa od razu lokalnie

Otwórz `index.html` w przeglądarce. Dane trzymane są w localStorage tego urządzenia.

## Krok 2 — wspólna baza dla całej brygady (Firebase, darmowe)

1. Wejdź na https://console.firebase.google.com i zaloguj się kontem Google.
2. **Utwórz projekt** (nazwa dowolna, np. `plan-dyzurow`; Google Analytics można wyłączyć).
3. W menu po lewej: **Build → Realtime Database → Create database**.
   Lokalizacja: `europe-west1` (Belgia). Tryb: **locked mode** (zaraz zmienimy reguły).
4. Zakładka **Rules** — wklej i opublikuj:
   ```json
   {
     "rules": {
       "dyzury": { ".read": true, ".write": true }
     }
   }
   ```
   Uwaga: to otwiera zapis każdemu, kto zna adres bazy — dla tablicy dyżurów to
   akceptowalne (PIN w aplikacji to miękka blokada), ale nie trzymaj tam nic wrażliwego.
5. Kliknij ikonę zębatki → **Project settings** → sekcja **Your apps** → dodaj
   aplikację **Web** (`</>`), nazwa dowolna, bez hostingu.
6. Skopiuj pokazany obiekt `firebaseConfig` i wklej go w `index.html`
   w miejscu `const FIREBASE_CONFIG = null;` (na górze sekcji `<script>`).
   Ważne: config musi zawierać pole `databaseURL` — jeśli go nie ma, skopiuj adres
   bazy z zakładki Realtime Database (np. `https://plan-dyzurow-default-rtdb.europe-west1.firebasedatabase.app`).
7. W stopce aplikacji pojawi się „Online · dane wspólne dla wszystkich".

## Krok 3 — publikacja pod jednym adresem

Najprościej: **Netlify Drop** — https://app.netlify.com/drop — przeciągnij folder
z `index.html` na stronę i dostajesz publiczny adres (np. `https://cos-tam.netlify.app`).
Ten adres wysyłasz brygadzie. Po każdej zmianie pliku wrzucasz go ponownie.

Alternatywy: GitHub Pages, Cloudflare Pages — też darmowe.

## Uwagi

- Zapis wygrywa „ostatni pisze" — zakładamy jednego brygadzistę edytującego naraz.
- Konfig Firebase w pliku HTML jest jawny z założenia (tak działają aplikacje webowe
  Firebase) — dostęp kontrolują reguły bazy, nie tajność konfigu.
