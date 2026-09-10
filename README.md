# LemurNet — wyszukiwarka + DNS lookup

Statyczna wyszukiwarka na GitHub Pages z automatycznym crawlerem (GitHub Actions)
i podglądem DNS przez publiczny resolver Cloudflare.

## Struktura repozytorium

```
index.html                        <- strona (frontend)
db.json                           <- baza stron (tytuł, opis) - nadpisywana przez bota
dns.json                          <- baza DNS (domena -> IP) - nadpisywana przez bota
seeds.json                        <- lista 100 domen do zaindeksowania
.github/workflows/bot.yml         <- harmonogram GitHub Actions
.github/workflows/crawl.js        <- właściwy crawler (strony + DNS)
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
5. **Zakładka Actions → LemurNet Indexer → Run workflow** — uruchom ręcznie
   pierwszy raz, żeby od razu zastąpić placeholdery w `db.json` i `dns.json`
   (100 domen z `seeds.json`) prawdziwymi danymi — realnymi tytułami stron
   i realnymi adresami IP pobranymi na żywo przez runner GitHub Actions
   (który, w przeciwieństwie do samej strony, ma pełny dostęp do sieci).
   Nie trzeba czekać 6 godzin na pierwszy automatyczny cykl.

## Jak działa wyszukiwanie na stronie

- Wpisujesz pełny adres z `http://` lub `https://` → przeglądarka od razu
  tam przechodzi.
- Wpisujesz gołą domenę (np. `wp.pl`) → strona sprawdza `dns.json` i pokazuje
  zapisany tam adres IP (to statyczny plik, nie żywe zapytanie DNS —
  aktualizowany co 6h przez bota).
- Wpisujesz dowolne hasło → przeszukiwana jest lokalna baza `db.json`.

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
