# 64_shades_of_Gray
[64_shades_of_Gray.bin](../../_resources/64_shades_of_Gray.bin)

Zadanie polega na tym, że plik jest w kodzie Graya a nie naturalnym kodzie binarnym. Po zmianie ciągu Graya na ciąg binarny powstanie dekodowalny ciąg znaków base64, który po odkodowaniu da flagę. 

Na przykładzie dużo krótszego pliku, którego `hex` wygląda tak:
```
00000000: 5664 fd47 5661 a3a3                      Vd.GVa..
```
w zapisie dwójkowym wygląda:
```
00000000: 01010110 01100100 11111101 01000111 01010110 01100001  Vd.GVa
00000006: 10100011 10100011                                      ..
```
po usunięciu adresów i reprezentacji ASCII:
```
0101011001100100111111010100011101010110011000011010001110100011
```
Gdy założymy że jest to w kodzie Graya i zmienimy tą liczbę na liczbę zakodowaną w naturalnym kodzie binarnym np. za pomocą narzędzia online (https://www.lambdatest.com/free-online-tools/gray-to-binary)

uzyskamy:
```
0110010001000111010101100111101001100100010000010011110100111101
```
co po zmianie do reprezentacji hex
```
01100100 01000111 01010110 01111010 01100100 01000001 00111101 00111101
   64       47       56       7a       64       41       3d       3d
```

i zmianie na plik za pomocą komendy `echo -n "6447567a64413d3d" | xxd -r -p > plik.bin`  

```
00000000: 6447 567a 6441 3d3d                      dGVzdA==
```

po zdekodowaniu base64: `cat plik.bin | base64 -d`, da wartość `test`.

Dłuższy ciąg w kodzie Graya jaki był w treści faktycznego zadania dodawał jedną dodatkową trudność. Był zbyt długi, żeby działały z tym ciągiem narzędzia online. Należało napisać własny skrypt do zamiany kodu Graya na kod binarny.

Jak zamienić liczbę w kodzie Gray na liczbę w kodzie binarnym i odwrotnie możemy dowiedzieć się z Wikipedii (https://pl.wikipedia.org/wiki/Kod_Graya).

`0x1753c = 0b10111010100111100`

Jeśli chcemy zamienić `10111010100111100` na kod Graya to musimy przesunąć liczbę w postaci binarnej o jeden bit w prawo, i wykonać operację `XOR` na odpowiednich bitach liczby i wyniku przesunięcia zamienianej liczby o jeden bit w prawo.

```
LICZBA      	10111010100111100 (w kodzie binarnym)
LICZBA >> 1 	01011101010011110
XOR
=  	        11100111110100010 (w kodzie GRAYa)
```

Jeśli chcemy zmienić `11100111110100010` na kod binarny to musimy przyjąć za pierwszy najbardziej znaczący bit kodu binarnego pierwszy najbardziej znaczący bit kodu Graya, a każdy kolejny bit obliczyć jako XOR kolejnego bitu z liczby zapisanej w kodzie Graya z poprzednio wyznaczonym bitem kodu binarnego. 
```
LICZBA      11100111110100010

Krok 01:    pierwszy bit przepisujemy 1 = 1 )
Krok 02:    1________________  (1 XOR 1 = 0 )
Krok 03:    10_______________  (0 XOR 1 = 1 )
Krok 04:    101______________  (1 XOR 0 = 1 )
Krok 05:    1011_____________  (1 XOR 0 = 1 )
Krok 06:    10111____________  (1 XOR 1 = 0 )
Krok 07:    101110___________  (0 XOR 1 = 1 )
Krok 08:    1011101__________  (1 XOR 1 = 0 )
Krok 09:    10111010_________  (0 XOR 1 = 1 )
Krok 10:    101110101________  (1 XOR 1 = 0 )
Krok 11:    1011101010_______  (0 XOR 0 = 0 )
Krok 12:    10111010100______  (0 XOR 1 = 1 )
Krok 13:    101110101001_____  (1 XOR 0 = 1 )
Krok 14:    1011101010011____  (1 XOR 0 = 1 )
Krok 15:    10111010100111___  (1 XOR 0 = 1 )
Krok 16:    101110101001111__  (1 XOR 1 = 0 )
Krok 17:    1011101010011110_  (0 XOR 0 = 0 )
Krok 18:    10111010100111100  (0           )
```

Rozwiązanie napisane w kodzie **python** w rozbiciu na poszczególne etapy jak we writeupie:
```python
import base64

def gray2bin(x: int) -> int:
    poprzedni_bit = (x >> (x.bit_length() - 1)) & 1
    wynik = poprzedni_bit
    for i in range(x.bit_length() - 2, -1, -1):
        bit = (x >> i) & 1
        nowy_bit = bit ^ poprzedni_bit
        wynik = (wynik << 1) | nowy_bit
        poprzedni_bit = nowy_bit
    return wynik


def from_file(filename: str) -> str:
    with open(filename, "rb") as file:
        byte_data = file.read()
    print(f"Odczytano dane z \"{filename}\":")
    print(byte_data)
    print(byte_data.hex())
    print(bin(int(byte_data.hex(), 16))[2:])
    return byte_data


# Program główny
# Krok 1: Wczytaj dane z pliku binarnego
filename = "64_shades_of_gray.bin"
byte_data = from_file(filename)

# Krok 2: Przekształć dane binarne z kodu Gray’a na kod binarny
hex_value = int(byte_data.hex(), 16)  # Z pliku jako liczba
gray_to_bin_value = gray2bin(hex_value)

# Wyświetl dane po konwersji z kodu Gray'a
bin_gray_to_bin = bin(gray_to_bin_value)[2:]
print(bin_gray_to_bin)
print(hex(gray_to_bin_value)[2:])

# Krok 3: Przekształć dane na Base64 i UTF-8
decoded_hex = bytes.fromhex(hex(gray_to_bin_value)[2:])
print(decoded_hex.decode("utf-8"))
base64_decoded = base64.b64decode(decoded_hex).decode("utf-8")
print(base64_decoded)

```