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
   Każde uruchomienie: przetwarza paczkę 200 domen z `seeds.json`, wyciąga
   z nich tytuł, opis i fragment realnej treści (do wyszukiwania pełnotekstowego),
   rozwiązuje ich IP, **oraz dopisuje do `seeds.json` nowe domeny znalezione
   jako linki na odwiedzonych stronach** (max 300 nowych na uruchomienie,
   twardy sufit 100 000 domen łącznie). Indeks rośnie sam, z realnych
   powiązań między stronami — tak jak robią to prawdziwe wyszukiwarki,
   tylko dużo mniejszym nakładem.

## Jak działa wyszukiwanie na stronie

- Wpisujesz pełny adres z `http://` lub `https://` → przeglądarka od razu
  tam przechodzi.
- Wpisujesz gołą domenę (np. `wp.pl`) → strona sprawdza `dns.json` i pokazuje
  zapisany tam adres IP.
- Wpisujesz dowolne hasło → przeszukiwane są tytuł, opis **i fragment
  realnej treści strony** (do 800 znaków), nie tylko metadane.

## Uwaga o rozmiarze plików (to jest realne ograniczenie, nie drobiazg)

Połączenie "duża liczba domen" + "pełny tekst każdej" rośnie w plik, którego
przeglądarka nie udźwignie bez ograniczeń — dlatego są dwa twarde limity:
fragment treści ucięty do 800 znaków i sufit 100 000 domen łącznie.
Nawet z tymi limitami, przy zbliżaniu się do pełna wypełnionego indeksu
`db.json` może ważyć **kilkadziesiąt megabajtów** — to zauważalne na słabym
łączu mobilnym. Jeśli po miesiącach wzrostu indeksu strona zacznie się
odczuwalnie dłużej ładować, rozwiązaniem jest podzielenie `db.json` na
mniejsze pliki ładowane na żądanie (np. alfabetycznie) zamiast jednego
wielkiego pliku — to osobna zmiana architektury, do zrobienia gdy realnie
będzie potrzebna, nie teraz na zapas.

## Uczciwe zastrzeżenie co do skali

To nie jest i nie będzie "cały internet" ani odpowiednik Google — Google
indeksuje setki miliardów podstron przy pomocy tysięcy serwerów. To, co tu
jest, to realny, samodzielnie rosnący indeks tej części sieci, do której
bot faktycznie dotrze podążając za linkami, w tempie ograniczonym limitami
bezpieczeństwa (żeby nie zostać zbanowanym jako podejrzany ruch) i rozmiarem
pliku, który przeglądarka jest w stanie sensownie obsłużyć.

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
