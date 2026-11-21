# DuckPCRE2: wrapper PCRE2 dla Ducklinga

Zespół: Pixel of Software (Marcin Kowalczyk). Temat: P02 DuckBridge, biblioteka PCRE2 (poziom 2). Skład: 1 osoba.

Moduł `regex` (`app/regex.dk`) udostępnia w Ducklingu wyrażenia regularne przez 8-bitowe API biblioteki C PCRE2:
* kompilację wzorca z błędem zawierającym kod, opis i pozycję;
* dopasowanie, odczyt całego dopasowania i grup przechwytujących;
* tryb UTF-8 ze sprawdzaniem tekstu.

Wyniki są kopiami: przesunięcia i teksty grup są przepisane do zwykłych wartości Ducklinga. Użytkownik nie dostaje wskaźnika do tablicy wyników PCRE2.

Całe minimum karty PCRE2 działa i jest sprawdzone testami. Do tego są dwa przykłady (podstawowy i praktyczna wyszukiwarka w logach) oraz rozszerzenia:
* wszystkie dopasowania z poprawną obsługą pustych dopasowań w UTF-8;
* grupy nazwane;
* opcje: bez wielkości liter, klasy Unicode, wielowierszowy, rozszerzony, zakotwiczony.

## Uruchomienie

Wymagane są:
* Python 3;
* Duckling (`duckc` w `PATH`, instalacja według tutoriala);
* biblioteka PCRE2 w odmianie 8-bitowej.

| System | Instalacja PCRE2 |
| --- | --- |
| Ubuntu | `sudo apt install libpcre2-8-0` |
| macOS | `brew install pcre2` |

Z katalogu projektu:

```bash
python3 build.py --target all --output build      # aplikacja, przykład i testy (wariant natywny)
python3 run.py --build build --target tests       # testy: PASS/FAIL, kod = liczba błędów
python3 run.py --build build --target przyklad    # przykład podstawowy
printf '^(\\S+) (\\S+) ERROR\\s+użytkownik=(\\S+) akcja=(\\S+)\ndane/serwer.log\n\n' | python3 run.py --build build --workdir .
```

Wyszukiwarka czyta trzy wiersze ze standardowego wejścia:
1. wzorzec;
2. plik;
3. opcje literami, gdzie pusty wiersz oznacza `u`:
   * `u` - UTF-8 ze sprawdzaniem;
   * `i` - bez wielkości liter;
   * `w` - klasy Unicode;
   * `x` - spacje we wzorcu ignorowane.

Dla każdego pasującego wiersza wypisuje grupy z pozycjami. Kody zakończenia:

| Kod | Znaczenie |
| --- | --- |
| 0 | są dopasowania |
| 1 | brak dopasowań |
| 2 | zły wzorzec lub opcje |
| 3 | błąd pliku |

`build.py` szuka biblioteki w tej kolejności:
1. ścieżka z opcji `--pcre2 /sciezka/libpcre2-8.so.0`;
2. `pkg-config libpcre2-8`;
3. typowe miejsca: `/usr/lib/x86_64-linux-gnu/libpcre2-8.so.0` oraz `/opt/homebrew/lib/libpcre2-8.dylib`;
4. `ctypes.util.find_library`.

`build.py` i `run.py` powstały z szablonu konkursu: doszła obsługa biblioteki i cel `przyklad`, a został tylko wariant natywny.

## Funkcje i demonstracja

| Funkcja z minimum karty | Sposób uruchomienia | Oczekiwany rezultat |
| --- | --- | --- |
| Kompilacja w 8-bitowym API; błąd z kodem, opisem i pozycją | przykład; wyszukiwarka ze wzorcem `akcja=(log` | `kod 114, 'missing closing parenthesis', pozycja 7`; w wyszukiwarce znacznik `^` pod pozycją 10 |
| Dopasowanie, brak dopasowania, zakres całego dopasowania i grup | przykład | `Pierwsze dopasowanie [13, 23): '2026-11-20'; rok '2026', miesiac '11', dzien '20'`; `Brak dopasowania: ok=true, znaleziono=false` |
| Grupy, które nie wzięły udziału w dopasowaniu | przykład: `(kot)\|(pies)` | `grupa 1 uczestniczy: false (pozycja -1), grupa 2: 'pies' na bajtach [11, 15)` |
| UTF-8 ze sprawdzaniem; pozycje w bajtach | przykład | `'ó' w 'Zażółć': bajty [4, 6)`; tekst z bajtem 0xFF: `kod -23, 'UTF-8 error: illegal byte (0xfe or 0xff)', bajt 2` |
| Kto zwalnia wzorzec i bufor; wyniki nie korzystają ze zwolnionej pamięci | przykład, testy | `Wyniki po zwolnieniu wzorcow nadal sa kopiami: '2026-11-20'` |
| Praktyczne zastosowanie | wyszukiwarka na `dane/serwer.log` | 2 wiersze ERROR z czterema grupami; wiersz 5 pominięty: niepoprawny UTF-8 na bajcie 38 |

**Demonstracja:**
1. Uruchom przykład.
2. Uruchom wyszukiwarkę z komendą powyżej. Wypisze dwa wiersze ERROR z datą, godziną, użytkownikiem (także „żaneta”) i akcją. Pozycje są w bajtach: grupa użytkownika zaczyna się na bajcie 38, bo „ż” w słowie „użytkownik” zajmuje 2 bajty.
3. Podaj wzorzec z błędem, np. `akcja=(log`. Dostaniesz kod 114 i znacznik pozycji.
4. Spróbuj opcji `ui` ze wzorcem `UŻYTKOWNIK=ŻANETA`. Polskie wielkie litery pasują do małych.

## API modułu `regex`

| Funkcja | Zwraca | Błąd |
| --- | --- | --- |
| `kompiluj(wzorzec: str, opcje: i64) -> Wzorzec` | skompilowany wzorzec (`ok`, `grupy` - liczba grup) | `ok == false`, `blad.kod` (np. 114), `blad.opis`, `blad.pozycja` (bajty we wzorcu) |
| `dopasuj(w: ref Wzorzec, tekst: str, start: i64) -> Dopasowanie` | pierwsze dopasowanie od bajtu `start` | `ok == false`: niepoprawny UTF-8 (kod −3..−23, `pozycjaBledu`), start w środku znaku (−36), start poza tekstem, zwolniony wzorzec |
| `wszystkie(w: ref Wzorzec, tekst: str) -> List[Dopasowanie]` | kolejne nienakładające się dopasowania | przy błędzie lista kończy się elementem z `ok == false` |
| `zwolnij(w: ref Wzorzec) -> bool` | zwalnia wzorzec i bufor dopasowania | `false`, gdy nie było czego zwalniać |
| `numerGrupy(w, nazwa) -> i64` | numer grupy `(?<nazwa>...)` | −1 dla nieznanej nazwy |
| `tekstGrupy(d, n) -> String` | tekst grupy (pusty, gdy grupa nie wzięła udziału) | - |
| `aktywny(w)`, `opisBledu(kod)`, `wersjaBiblioteki()` | pomocnicze | - |

**Opcje:** `BEZ_OPCJI`, `UTF8`, `BEZ_WIELKOSCI_LITER`, `KLASY_UNICODE`, `WIELOLINIOWY`, `KROPKA_WSZYSTKO`, `ROZSZERZONY`, `ZAKOTWICZONY`, łączone operatorem `|`. Inne opcje wrapper odrzuca kodem `ZLE_OPCJE`.

`Dopasowanie` ma pola:
* `ok` - czy nie było błędu;
* `znaleziono` - czy wzorzec pasuje;
* `grupy` - `List[Grupa]`, gdzie element 0 to całe dopasowanie, a lista ma zawsze 1 + liczba grup elementów;
* `kodBledu`, `blad`, `pozycjaBledu` - opis błędu.

`Grupa` ma pola `uczestniczy`, `poczatek`, `koniec` (w bajtach, zakres półotwarty) i `tekst`.

## Pozycje w bajtach i grupy bez udziału

**Wszystkie pozycje są liczone w bajtach UTF-8, a nie w znakach.** Polska litera zajmuje 2 bajty, więc w tekście „Zażółć”:

| Znak | Bajty |
| --- | --- |
| „Z” | 0 |
| „a” | 1 |
| „ż” | 2-3 |
| „ó” | 4-5 |

Dopasowanie „ó” ma zatem zakres [4, 6), choć „ó” to 4. znak. Zakresy są półotwarte: `koniec` wskazuje bajt za ostatnim. Długość fragmentu w bajtach to `koniec − poczatek`.

W trybie UTF-8 kropka `.` dopasowuje cały znak, czyli 2 bajty polskiej litery. Bez trybu UTF-8 dopasowuje jeden bajt. Start w środku znaku UTF-8 jest błędem `-36`.

Grupa, która nie wzięła udziału w dopasowaniu, ma `uczestniczy == false`, `poczatek == koniec == -1` i pusty tekst; PCRE2 oznacza ją wartością `PCRE2_UNSET`. To co innego niż grupa, która dopasowała pusty fragment. Na przykład w `(x)?(y)(z?)` dla tekstu „y”:
* grupa 1 nie wzięła udziału;
* grupa 3 uczestniczy z pustym zakresem [1, 1).

Wrapper zawsze zwraca wpis dla każdej grupy wzorca, także dla końcowych grup bez udziału, których PCRE2 nie liczy w wyniku `pcre2_match`.

## Pamięć: kto co zwalnia

Wywołanie `kompiluj` tworzy dwa obiekty biblioteki:
* skompilowany wzorzec (`pcre2_compile_8`);
* bufor dopasowania (`pcre2_match_data_create_from_pattern_8`), używany przez kolejne wywołania `dopasuj`.

Oba należą do obiektu `Wzorzec`. Wywołujący zwalnia je raz funkcją `zwolnij(&wzorzec)`, która wywołuje `pcre2_match_data_free_8` i `pcre2_code_free_8`. Po zwolnieniu:
* `dopasuj` zwraca błąd `ZWOLNIONY`;
* ponowne `zwolnij` zwraca `false`.

Wzorca nie należy kopiować przez `copy`, bo kopia wskazywałaby ten sam obiekt PCRE2.

Wyniki (`Dopasowanie`, `Grupa`) to kopie: wrapper od razu przepisuje przesunięcia z wektora wyników i wycina teksty grup do `String`. Kolejne dopasowanie nadpisuje bufor PCRE2, a zwolnienie wzorca go usuwa, ale wcześniejsze wyniki zostają nienaruszone; testy to sprawdzają. Tekst i wzorzec są przekazywane z jawną długością, więc mogą zawierać bajt NUL. Testy sprawdzają też 2000 cykli kompilacja, dopasowanie, zwolnienie.

## Środowisko i zależności

| Element | Wartość |
| --- | --- |
| Kompilator | `duckc 0.3-alpha (83882805a 2026-11-20)`, `commit-hash: 83882805a` |
| Uruchomienie | natywnie |
| System operacyjny | macOS 27.2, Apple Silicon (arm64) |
| Zależności | PCRE2 10.45 (Homebrew, `/opt/homebrew/lib/libpcre2-8.dylib`), Python 3 |

Wersja odniesienia z karty to PCRE2 10.42. Użyte funkcje 8-bitowego API i stałe (opcje, kody błędów, `PCRE2_UNSET`) mają w 10.45 te same deklaracje i wartości; sprawdziłem je w nagłówku `pcre2.h`. Projekt sprawdziłem na 10.45. Na Linuksie x86_64 (Ubuntu 24.04, `libpcre2-8.so.0`) go nie sprawdzałem. JIT nie jest używany.

## Struktura i działanie

| Plik | Odpowiedzialność |
| --- | --- |
| `app/regex.dk` | wrapper: deklaracje C, opcje, kompilacja, dopasowanie, kopiowanie grup, UTF-8, zwalnianie |
| `app/app.dk` | praktyczne zastosowanie: wyszukiwarka w logach z grupami |
| `app/pliki.dk` | odczyt pliku przez `fopen`/`fread`/`fclose` |
| `przyklady/podstawowy.dk` | przykład podstawowy: grupy, brak dopasowania, UTF-8, błędy |
| `tests/testy.dk` | testy grup, braku dopasowania, niepoprawnych wzorców, UTF-8 i pamięci |
| `dane/serwer.log` | przykładowy log z polskimi literami i jednym wierszem z niepoprawnym UTF-8 |

**Deklaracje C** są ręczne, według `pcre2.h`, z sufiksem `_8` (odmiana 8-bitowa). Użyte funkcje:
* minimalne API z karty: `pcre2_compile_8`, `pcre2_match_data_create_from_pattern_8`, `pcre2_match_8`, `pcre2_get_ovector_pointer_8`, `pcre2_get_error_message_8`, `pcre2_code_free_8`, `pcre2_match_data_free_8`;
* dodatkowo: `pcre2_get_startchar_8`, `pcre2_pattern_info_8`, `pcre2_substring_number_from_name_8`, `pcre2_config_8`.

**Uchwyty i NULL.** Wskaźniki `pcre2_code*` i `pcre2_match_data*` są przechowywane jako `u64`, więc NULL to 0. Konteksty przekazujemy jako 0, czyli NULL. W ABI 64-bitowych systemów (LP64) wskaźnik i `u64` przekazuje się identycznie, więc adapter C nie jest potrzebny. Na DVM byłby wymagany, dlatego wariant DVM nie jest obsługiwany.

**Błąd UTF-8 w tekście.** `pcre2_match_8` zwraca kod od −3 do −23, a pozycję pierwszego błędnego znaku wrapper odczytuje przez `pcre2_get_startchar_8`, zgodnie z dokumentacją `pcre2api`. Pierwszy element wektora wyników nie zawiera wtedy tej pozycji.

**Wszystkie dopasowania.** Po pustym dopasowaniu szukanie przesuwa się o jeden znak: w trybie UTF-8 o cały znak, nie o bajt. Dzięki temu pętla zawsze się kończy i nie zaczyna w środku polskiej litery.

**Ograniczenia:**
* jeden wzorzec to jeden bufor dopasowania, więc obiektu `Wzorzec` nie należy używać równolegle z wielu wątków;
* brak zamiany (`pcre2_substitute`);
* brak JIT;
* brak dopasowań częściowych.

## Sprawdzenie projektu

```bash
python3 run.py --build build --target tests      # 37 sprawdzeń
python3 run.py --build build --target przyklad   # kod 0 = wszystkie oczekiwania przykładu spełnione
```

Testy sprawdzają:
* niepoprawne wzorce z kodem i pozycją: `abc(def` → 114 na pozycji 7, `a{3,1}` → 104 na pozycji 5, `[z-a]` → 108 na pozycji 3, niepoprawny UTF-8 we wzorcu na pozycji 1;
* grupy z zakresami, dopasowanie od pozycji startowej i wszystkie dopasowania;
* brak dopasowania (to nie błąd);
* grupy bez udziału, także końcowe, oraz grupy z pustym dopasowaniem;
* grupy nazwane;
* pozycje w bajtach dla polskich liter oraz `.` w trybie UTF-8 i bez niego;
* wielkość liter i `\w` dla polskich liter;
* niepoprawny i ucięty UTF-8 w tekście z pozycją, start w środku znaku, bajt 0xFF bez trybu UTF-8;
* puste dopasowania;
* niezależność wyników od kolejnych dopasowań i zwolnienia;
* użycie i podwójne zwolnienie po zwolnieniu;
* bajt NUL;
* 2000 cykli kompilacja, dopasowanie, zwolnienie.

Scenariusze są w [tests/scenariusze.md](tests/scenariusze.md).

## Autorstwo i materiały

Kod i dokumentacja: Pixel of Software (Marcin Kowalczyk), z pomocą asystenta AI. Szczegóły są w [ZRODLA.md](ZRODLA.md).

Licencja własnego kodu: MIT (plik `LICENSE`). Biblioteka PCRE2 ma licencję BSD-3-Clause z wyjątkiem opisanym w jej pliku LICENCE i nie jest dołączona do repozytorium; wrapper łączy się z nią dynamicznie.
