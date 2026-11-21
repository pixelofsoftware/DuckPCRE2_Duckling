# Autorstwo i źródła

## Wkład autorów

| Autor | Wykonane elementy | Lokalizacja w projekcie |
| --- | --- | --- |
| Marcin Kowalczyk (Pixel of Software) | wybór biblioteki, założenia, przegląd i akceptacja kodu, publikacja | całość |
| Asystent AI Claude (Anthropic) | wrapper, przykłady, testy, przykładowy log, skrypty budowania, dokumentacja | `app/`, `przyklady/`, `tests/`, `dane/`, `build.py`, `run.py`, `README.md` |

## Wykorzystane materiały

| Materiał | Autor i źródło | Wersja lub data | Licencja | Sposób wykorzystania |
| --- | --- | --- | --- | --- |
| PCRE2 | Philip Hazel i współtwórcy, https://github.com/PCRE2Project/pcre2 | 10.45 (odniesienie z karty: 10.42) | BSD-3-Clause z wyjątkiem opisanym w LICENCE (https://github.com/PCRE2Project/pcre2/blob/pcre2-10.42/LICENCE) | biblioteka łączona dynamicznie, nie jest dołączona; deklaracje funkcji `_8` i stałe przepisane ręcznie z `pcre2.h` |
| Dokumentacja `pcre2api` | PCRE2 | 10.45 | jak biblioteka | sposób odczytu pozycji błędu UTF-8 (`pcre2_get_startchar`) |
| `build.py`, `run.py`, `LICENSE` | Fundacja Duckling, szablon projektu z materiałów na start DuckQuest | 2026 | MIT (plik `LICENSE`) | skrypty rozbudowane o dołączanie PCRE2 i cel `przyklad`; licencja z dopisanym autorem |
| `app/pliki.dk` | własny moduł z projektu DuckFolio (Pixel of Software) | 2026 | MIT | odczyt pliku przez `fopen`/`fread` |
| Duckling (kompilator i biblioteka standardowa) | ducktype-org/duckling | 0.3-alpha, commit 83882805a | licencja Ducklinga | zależność budowania |

`dane/serwer.log` to fikcyjny log przygotowany dla tego projektu (z celowo niepoprawnym UTF-8 w wierszu 5).

## Narzędzia AI

Kod wrappera, aplikacji, przykładu i testów, przykładowy log, zmiany skryptów budowania oraz dokumentację przygotowano z pomocą asystenta AI Claude (Anthropic). Autor ustalił zakres i wybór biblioteki, przejrzał wyniki, sprawdził działanie (budowa, 37 testów, przykład, wyszukiwarka) i odpowiada za oddaną wersję.

## Licencja własnego projektu

MIT, treść w pliku `LICENSE` (szablon Fundacji Duckling z dopisaną linią `Copyright (c) 2026 Pixel of Software (Marcin Kowalczyk)`). Licencja PCRE2 dotyczy samej biblioteki, instalowanej osobno.
