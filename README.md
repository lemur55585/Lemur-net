# LemurNet — wyszukiwarka + DNS lookup

Statyczna wyszukiwarka na GitHub Pages z automatycznym crawlerem (GitHub Actions)
i podglądem DNS przez publiczny resolver Cloudflare.

## Struktura repozytorium

```
index.html                        <- strona (frontend)
db.json                           <- baza stron (tytuł, opis) - nadpisywana przez bota
dns.json                          <- baza DNS (domena -> IP) - nadpisywana przez bota
seeds.json                        <- lista 10 000 domen (ranking Majestic)
.github/workflows/bot.yml         <- harmonogram GitHub Actions
.github/workflows/crawl.js        <- crawler wsadowy (strony + DNS)
.github/workflows/crawl_state.json <- offset, który śledzi którą paczkę domen przetworzyć następnym razem
```

## Wdrożenie krok po kroku

1. **Nowe repozytorium** na GitHubie (publiczne — prywatne też działa, ale
   ma limit minut Actions w darmowym planie).
2. Wgraj do niego wszystkie pliki z tego paczki, zachowując strukturę folderów
   (`.github/workflows/` musi zostać w tym miejscu).
3. **Ustawienia → Actions → General → Workflow permissions** — zaznacz
   *„Read and write permissions”* i zapisz. Bez tego bot nie będzie mógł
   wypchnąć zaktualizowanego `db.json` (błąd `403` przy `git push`).
4. **Ustawienia → Pages** — Source: `Deploy from a branch`, branch: `main`,
   folder: `/ (root)`. Zapisz — strona pojawi się pod
   `https://<twoj-login>.github.io/<nazwa-repo>/`.
5. **Zakładka Actions → LemurNet Indexer → Run workflow** — uruchom ręcznie.
   Ważne: przy 10 000 domen **jedno uruchomienie przetwarza tylko paczkę
   200 domen** (offset zapisany w `crawl_state.json`), nie całość naraz.
   Powód: odpytanie 10 tys. niezwiązanych ze sobą serwerów jednocześnie
   wygląda jak atak i kończy się banowaniem IP runnera. Pełne odświeżenie
   całej bazy to ok. 50 uruchomień — przy cron co 6h to ok. 12-13 dni na
   pełny cykl. Żeby przyspieszyć pierwsze wypełnienie danych, możesz klikać
   "Run workflow" ręcznie kilka razy pod rząd (każde kolejne przesuwa offset
   i robi następną paczkę) zamiast czekać na automatyczny harmonogram.

## Jak działa wyszukiwanie na stronie

- Wpisujesz pełny adres z `http://` lub `https://` → przeglądarka od razu
  tam przechodzi.
- Wpisujesz gołą domenę (np. `wp.pl`) → strona sprawdza `dns.json` i pokazuje
  zapisany tam adres IP (to statyczny plik, aktualizowany paczkami co 6h,
  nie żywe zapytanie DNS).
- Wpisujesz dowolne hasło → przeszukiwana jest lokalna baza `db.json`.

## Uwaga o rozmiarze plików

`db.json` przy 10 000 wpisów waży ok. 2,3 MB, `dns.json` ok. 0,6 MB —
przeglądarka pobiera je w całości przy wejściu na stronę. To akceptowalne
na desktopie i dobrym łączu mobilnym, ale zauważalne opóźnienie na słabym
Wi-Fi/LTE. Jeśli to będzie problem, da się to przyspieszyć (np. dzieląc
bazę na fragmenty ładowane na żądanie) — daj znać, jeśli tego potrzebujesz.

## Dodawanie nowych stron do indeksu

Edytuj `seeds.json` (jedna domena = jeden wpis w tablicy), zacommituj —
przy najbliższym uruchomieniu bota (ręcznym albo co 6h) strona zostanie
odwiedzona i dopisana do `db.json` z prawdziwym tytułem i opisem.

## Uwagi eksploatacyjne

- GitHub potrafi opóźniać lub pomijać harmonogramy (`cron`) w repozytoriach
  bez świeżej aktywności — jeśli indeks długo się nie odświeża, po prostu
  uruchom workflow ręcznie.
- Crawler nie usuwa wpisu, jeśli strona chwilowo nie odpowiada (timeout/500) —
  usuwane są tylko domeny ręcznie skasowane z `seeds.json` przy kolejnym
  pełnym odświeżeniu, jeśli sam to dopiszesz do logiki; obecnie stare wpisy
  zostają, dopóki nie podmienisz ich ręcznie w `db.json`.
