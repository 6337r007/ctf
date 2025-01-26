# unboxing

[x.nthw](./assets/x.nthw)

## Rozpakowywanie

Po ściągnięciu pliku zadania należy sprawdzić co to jest za rodzaj pliku: komenda `file x.nthw` odkrywa przed nami informację że jest to `x.nthw: XZ compressed data, checksum CRC64` więc możemy zmienić rozszerzenie pliku na `.xy` za pomocą komendy `mv x.nthw x.xz`, oraz za pomocą komendy `xz -d x.xz` przeprowadzić dekompresję pliku `x.xz`.
Po tej operacji powstaje plik `x` który po sprawdzeniu za pomocą `file x` okazuje się być `x: bzip2 compressed data, block size = 100k`, po dekompresji za pomocą `bzip2 -d x` otrzymujemy `x.out`.
Po sprawdzeniu za pomocą `file x.out`, wiemy że jest to `x.out: zlib compressed data`, po dekompresji za pomocą `zlib-flate -uncompress < x.out > flaga` otrzymujemy plik `flaga`.
Po sprawdzeniu za pomocą `file flaga`, dowiadujemy się że jest to `flaga: Zip archive data, at least v5.1 to extract, compression method=AES Encrypted`, czyli zabezpieczony hasłem plik `.zip`. Dla porządku jeśli mamy takie preferencje możemy dodać rozszerzenie do pliku `mv flaga flaga.zip`

### Alternatywnie 

możemy użyć komendy `binwalk -eM x.nthw` która gdy znajdzie skompresowany plik to go dekompresuje i znów sprawdza czy zawiera się w nim jakiś skompresowany plik i tak aż do momentu gdy nie jest w stanie przeprowadzić dekompresji lub nie znajdzie skompresowanego pliku.  Wynikiem działania komendy będzie struktura jak niżej:

```
.
├── x.nthw
└── _x.nthw.extracted
    ├── 0
    ├── _0.extracted
    │   ├── 0
    │   └── _0.extracted
    │       ├── 0
    │       ├── _0.extracted
    │       │   └── 0.zip
    │       └── 0.zlib
    ├── 0.xz
    ├── 1B
    └── _1B.extracted
        ├── 0
        ├── _0.extracted
        │   └── 0.zip
        └── 0.zlib

7 directories, 11 files
```
 możemy wtedy  sprawdzić plik `0.zip` i wyjdzie nam to samo, że jest on zabezpieczony hasłem `_x.nthw.extracted/_0.extracted/_0.extracted/_0.extracted/0.zip: Zip archive data, at least v5.1 to extract, compression method=AES Encrypted`. 
Zmieniamy mu nazwę na `flaga.zip` i możemy przejść do łamania hasła. 

## Łamanie hasła pliku .zip

Najprostszą metodą aby wyciągnąć z tego pliku hash hasła i zapisać do pliku o nazwie `hash.john` jest komenda `zip2john flaga.zip > hash.john`.

Teraz można rozpocząć łamanie hasha hasła metodą słownikową za pomocą komendy `john --wordlist=~/rockyou.txt hash.john`.

W moim przypadku dla wygody w folderze domowym mam rozpakowany słownik `rockyou.txt`.  Jeśli również chcesz mieć ten słownik pod ręką to możesz to zrobić kopiując go do folderu w którym się znajdujesz i rozpakowując za pomocą sekwencji dwóch komend `cp /usr/share/wordlist/rockyou.txt.gz . && gunzip rockyou.txt.gz`, a następnie przystąpić do łamania hasha za pomocą tym razem `john --wordlist=rockyou.txt hash.john`, ponieważ słownik znajduje się w tym samym folderze w którym akurat pracujesz.

Po niecałych 2 minutach poznajemy hasło dzięki któremu możemy rozpakować plik `flaga.zip`. 

### Alternatywnie, 
możemy do łamania hasha użyć programu `hashcat`, który jeśli jest poprawnie skonfigurowany to do łamania hasha użyje naszej karty graficznej, natomiast trzeba przygotować plik z hashem wg wytycznych jakie daje producent tego programu (https://hashcat.net/wiki/doku.php?id=example_hashes).

| Hash-Mode | Hash-Name | Example |
|-|-|-|
| 13600 | WinZip | `$zip2$*0*3*0*e3222d3b65b5a2785b192d31e39ff9de*1320*e*19648c3e063c82a9ad3ef08ed833*3135c79ecb86cd6f48fc*$/zip2$` |

Czyli hash z pliku `hash.john` dostosować do wymagań w edytorze tekstu i z postaci:

`flaga.zip/flaga.txt:$zip2$*0*3*0*c03283f3fbbae48ffacdbf7f8aae4b32*1b21*33*0baaa9baee318a563bb8cb5caf375860d709232554aaa9261df313de38e66710a6476d4e134bb9eb47d2b2ee1721ebba112e2f*466bf854b7497c548e2d*$/zip2$:flaga.txt:flaga.zip:flaga.zip`, 

dostosować do postaci:

`$zip2$*0*3*0*c03283f3fbbae48ffacdbf7f8aae4b32*1b21*33*0baaa9baee318a563bb8cb5caf375860d709232554aaa9261df313de38e66710a6476d4e134bb9eb47d2b2ee1721ebba112e2f*466bf854b7497c548e2d*$/zip2$`,

następnie zapisać w pliku `hash.hashcat`.

Wtedy za pomocą komendy `hashcat -m 13600 -a 0 -o pass hash.hashcat ~/rockyou.txt` po około 30 sekundach hasło również zostanie złamane i tym razem zapisane do pliku `pass`.

## Zakończenie

Plik możemy rozpakować np. za pomocą komendy `7z x flaga.zip` wprowadzając w odpowiednim momencie poznane hasło, a flagę wyświetlić za pomocą komendy `cat flaga.txt`.
