# Scenariusze sprawdzenia

Próby z 2026-11-21 na macOS 27.2 (Apple Silicon), `duckc 0.3-alpha (83882805a 2026-11-20)`, PCRE2 10.45 (Homebrew), wariant natywny, po `python3 build.py --target all --output build`.

| Scenariusz | Dane lub czynności | Oczekiwany wynik | Komenda lub procedura | Otrzymany wynik |
| --- | --- | --- | --- | --- |
| Zwykłe działanie: grupy | `(\w+)@(\w+)\.(pl\|com)` na „kontakt: jan@firma.pl, ola@szkola.com” | grupa 0 [9, 21), grupy [9, 12) „jan”, [13, 18) „firma”, [19, 21) „pl”; 2 dopasowania | `run.py --target tests` | zgodny - OK |
| Brak dopasowania | tekst bez adresów | `ok = true`, `znaleziono = false`, brak grup | testy, przykład | zgodny - OK |
| Grupy bez udziału | `(a)\|(b)` na „xb”; `(x)?(y)(z?)` na „y”; `(a)(b)?` na „a” | grupa bez udziału: −1 i pusty tekst; grupa pusta uczestniczy; lista zawsze 1 + liczba grup | testy | zgodny - OK |
| Niepoprawny wzorzec | `abc(def`, `a{3,1}`, `[z-a]`, niepoprawny UTF-8 we wzorcu | kody 114, 104, 108 i ujemny; pozycje 7, 5, 3, 1; opisy | testy, przykład, wyszukiwarka (`akcja=(log`: kod 114, pozycja 10) | zgodny - OK |
| UTF-8: pozycje w bajtach | „ó” w „Zażółć”; `^..` na „żółw” z UTF-8 i bez | [4, 6); 4 bajty z UTF-8, 2 bajty bez | testy, przykład | zgodny - OK |
| Niepoprawny UTF-8 w tekście | bajt 0xFF; ucięta litera na końcu; start w środku znaku | kod −3..−23 z opisem i pozycją 2 / 3; kod −36 | testy, przykład | „UTF-8 error: illegal byte (0xfe or 0xff)”, bajt 2 - OK |
| Wielkość liter i klasy Unicode | „żółw” bez wielkości liter na „ŻÓŁW”; `\w+` z klasami Unicode na „  źdźbło!” | dopasowanie [0, 7); [2, 11) „źdźbło” | testy | zgodny - OK |
| Puste dopasowania | `""` na „żb” w UTF-8; `x*` na „axxb” | pozycje 0, 2, 3; 4 dopasowania; pętla się kończy | testy | zgodny - OK |
| Pamięć | wynik po kolejnym dopasowaniu i po zwolnieniu; użycie po zwolnieniu; podwójne zwolnienie; 2000 cykli | wyniki nienaruszone; błąd `ZWOLNIONY`; `false`; brak awarii | testy | zgodny - OK |
| Bajt NUL | NUL we wzorcu i w tekście | dopasowanie [2, 5) | testy | zgodny - OK |
| Praktyczne zastosowanie | wyszukiwarka, `dane/serwer.log`, wiersze ERROR | wiersze 3 i 6 z 4 grupami; wiersz 5 pominięty (UTF-8, bajt 38) | `printf '…' \| python3 run.py --build build --workdir .` | zgodny, kod 0 - OK |
| Testy | 37 sprawdzeń | `Bledy: 0`, kod 0 | `python3 run.py --build build --target tests` | `Bledy: 0` - OK |
| Odtworzenie w nowym katalogu | świeży klon repozytorium | budowa, przykład, wyszukiwarka i testy zgodne z README | `git clone`, komendy z README | macOS: OK; Linux x86_64 (Ubuntu 24.04, PCRE2 10.42): nie sprawdzono |
