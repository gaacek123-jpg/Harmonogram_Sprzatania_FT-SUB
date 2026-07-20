# Plan dyżurów · STREFA FT / SUB

Aplikacja do układania harmonogramu sprzątania dla brygady. Cała w jednym pliku:
[`index.html`](index.html) — vanilla JS, bez builda, bez zależności.

**Na żywo:** https://gaacek123-jpg.github.io/Harmonogram_Sprzatania_FT-SUB/
Ten adres dostaje brygada. Dane są wspólne — każdy widzi ten sam plan.

## Tryby

- **Tydzień** — tabela zadania × dni z przypisanymi osobami, przycisk Drukuj/PDF (układ poziomy A4).
- **Pracownik** — każdy klika w siebie i widzi: dziś, najbliższe, ostatnie dyżury (z kim, czy z wózkiem).
- **Brygadzista** (opcjonalny PIN w ustawieniach):
  - siatka **dyspozycyjności** na 7 dni (✓ / —), kropki statusu przy datach,
  - **Plany tygodnia** — układanie dnia jednym klikiem albo „Ułóż brakujące" na cały tydzień (wg zaznaczonej dyspo),
  - edytor planu: zmiana osób (przy nazwisku widać obłożenie w tygodniu), „Ktoś wypadł? → Zdejmij i zastąp" (odznacza też dyspo),
  - **Obłożenie** — paski na ten tydzień + suma z całej historii,
  - ustawienia: brygada (z uprawnieniami na wózki), zadania (waga, liczba osób, częstotliwość, stała osoba, „razem z"), dni robocze, PIN, reset.

**Motyw jasny / ciemny** — przełącznik w prawym górnym rogu (zapamiętywany, przy pierwszym
wejściu dopasowuje się do ustawień systemu). Jasny w barwach FlexLink (czerwień + grafit),
ciemny w różu/fiolecie na graficie.

Rotacją steruje balans: suma wag z historii + obciążenie danego dnia + świeżość
(kto robił zadanie niedawno, dostaje je później).

## Dane i synchronizacja

- Wspólna baza to **Firebase Realtime Database** (projekt `plan-dyzurow`), dane pod kluczem `dyzury/state`.
- Config jest wpisany w `index.html` (stała `FIREBASE_CONFIG`) — patrz sekcja *Bezpieczeństwo*.
- Bez połączenia z siecią / bez configu aplikacja i tak działa **lokalnie** (localStorage tego urządzenia).
- Zapis działa na zasadzie „ostatni wygrywa" — zakładamy jednego brygadzistę edytującego naraz.

## Hosting i wdrażanie

Strona stoi na **GitHub Pages** z gałęzi `main` (folder główny repo). Publikacja jest automatyczna:

```
# po edycji index.html
git add index.html
git commit -m "opis zmiany"
git push origin main
```

Po ~40–60 s nowa wersja jest pod tym samym adresem. Na telefonie warto zrobić twarde
odświeżenie (Ctrl+F5), bo Pages potrafi chwilę trzymać starą wersję w cache.

> Uwaga na dwa źródła zmian: jeśli edytujesz plik **przez stronę GitHuba**, a równolegle
> ktoś pracuje **lokalnie**, wersje się rozjeżdżają i push wymaga scalenia (`git pull` najpierw).
> Najczyściej: zmiany idą jedną drogą.

## Bezpieczeństwo

- **Klucz API Firebase w kodzie jest jawny z założenia** — tak działają webowe aplikacje Firebase,
  klucz tylko wskazuje projekt, nie chroni danych. GitHub może go oznaczyć w „secret scanning";
  to nie jest wyciek, klucza nie trzeba rotować (można zamknąć alert jako *Won't fix*).
- Realnym dostępem steruje **reguły bazy**, które są świadomie **otwarte** (`.read`/`.write: true`).
  Czyli każdy, kto zna adres bazy, może odczytać i nadpisać harmonogram. Dla tablicy dyżurów
  (imiona/inicjały + zadania, zero danych wrażliwych) to akceptowalny kompromis. **Nie trzymaj tam
  nic poufnego.** PIN brygadzisty to miękka blokada UI, nie zabezpieczenie.
- Ewentualne utwardzenie (opcjonalne): ograniczenie klucza do domeny `gaacek123-jpg.github.io`
  w Google Cloud, albo warunek przy zapisie w regułach bazy.

## Odtworzenie bazy Firebase od zera (referencyjnie)

Potrzebne tylko przy zakładaniu nowego projektu (np. inny właściciel bazy):

1. https://console.firebase.google.com → **Utwórz projekt** (Analytics można wyłączyć).
2. **Build → Realtime Database → Create database**, lokalizacja `europe-west1`, tryb *locked*.
3. Zakładka **Rules** — wklej i opublikuj:
   ```json
   { "rules": { "dyzury": { ".read": true, ".write": true } } }
   ```
4. Zębatka → **Project settings → Your apps** → dodaj aplikację **Web** (`</>`), bez hostingu.
5. Skopiuj `firebaseConfig` i wklej w `index.html` w miejsce stałej `FIREBASE_CONFIG`
   (nazwa zmiennej musi zostać `FIREBASE_CONFIG`). Config musi mieć pole `databaseURL`.
6. W stopce aplikacji pojawi się „Online · dane wspólne dla wszystkich".

## Licencja

MIT — patrz [LICENSE](LICENSE).
