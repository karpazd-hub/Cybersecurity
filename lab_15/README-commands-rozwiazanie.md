# Laboratorium 15 — rozwiązanie

> Uwaga: wartości oznaczone `‹…›` (wersja OpenSSL, skróty SHA-256, dane certyfikatu) pochodzą z mojego terminala — wpisuję je po uruchomieniu komend, bo na każdym komputerze będą inne.

## Przygotowanie środowiska

1. Przejdź do katalogu **content/15/lab**.

cd content/15/lab
```

2. Potwierdź, że pracujesz we właściwej lokalizacji.

pwd
```

3. Wyświetl pliki i katalogi dostępne w laboratorium.

ls -la
```

4. Sprawdź, czy OpenSSL jest dostępny.

which openssl
```

5. Zanotuj wersję OpenSSL w szablonie notatek.

openssl version
```
Wersja: `‹np. OpenSSL 3.x.x›`

6. Utwórz katalog **work**, jeżeli jeszcze nie istnieje.

mkdir -p work
```
(`-p` – nie zgłasza błędu, jeśli katalog już istnieje)

7. Skopiuj **materials/sample-secret.txt** do katalogu roboczego jako **secret.txt**.

cp materials/sample-secret.txt work/secret.txt
```

8. Skopiuj **materials/lab-notes-template.md** do katalogu roboczego jako **lab-notes.md**.

cp materials/lab-notes-template.md work/lab-notes.md
```

9. Przejdź do katalogu **work**.

cd work
```

10. Potwierdź, że oba pliki istnieją i są czytelne.

ls -l secret.txt lab-notes.md
test -r secret.txt && test -r lab-notes.md && echo "Oba pliki są czytelne"
```
-rw-rw-r-- 1 kali kali 166 Oct  5 12:39 lab-notes.md
-rwxrwx--- 1 kali kali 219 Oct  5 12:40 secret.txt
Oba pliki są czytelne

11. Otwórz **secret.txt** i upewnij się, że zawiera wyłącznie testową wiadomość.

cat secret.txt
```
To jest testowa poufna wiadomość do laboratorium z kryptografii.
Nie zawiera prawdziwych sekretów, haseł ani danych osobowych.
Cel: sprawdzić kodowanie, hashowanie, szyfrowanie symetryczne, RSA i model hybrydowy.
---

## 1. Kodowanie to nie szyfrowanie

1. Odczytaj zawartość **secret.txt**.


```

2–3. Zakoduj zawartość pliku przy użyciu Base64 i obejrzyj wynik bez zapisywania.

base64 secret.txt
```
VG8gamVzdCB0ZXN0b3dhIHBvdWZuYSB3aWFkb21vxZvEhyBkbyBsYWJvcmF0b3JpdW0geiBrcnlw
dG9ncmFmaWkuCk5pZSB6YXdpZXJhIHByYXdkeml3eWNoIHNla3JldMOzdywgaGFzZcWCIGFuaSBk
YW55Y2ggb3NvYm93eWNoLgpDZWw6IHNwcmF3ZHppxIcga29kb3dhbmllLCBoYXNob3dhbmllLCBz
enlmcm93YW5pZSBzeW1ldHJ5Y3puZSwgUlNBIGkgbW9kZWwgaHlicnlkb3d5LgoK

4. Zapisz zakodowaną reprezentację jako **secret.base64.txt**.

base64 secret.txt > secret.base64.txt
```

5. Otwórz zapisany plik i oceń, czy oryginalny tekst jest bezpośrednio czytelny.

cat secret.base64.txt
```
Tekst nie jest bezpośrednio czytelny, ale to tylko inna reprezentacja tych samych bajtów.

6. Odkoduj **secret.base64.txt**.

base64 -d secret.base64.txt
```

7. Porównaj odkodowaną treść z pierwotnym **secret.txt**.

diff <(base64 -d secret.base64.txt) secret.txt && echo "Identyczne"
```
Wynik: Identyczne

8. Sprawdź, czy odkodowanie wymagało hasła, klucza albo innego sekretu.
Nie wymagało — komenda `base64 -d` nie przyjmuje żadnego hasła ani klucza.

### Notatki
- Wynik kodowania Base64: `‹zawartość secret.base64.txt›`
- Base64 nie wymaga sekretu.
- Base64 nie zapewnia poufności — każdy może to odkodować.
- Kodowanie zmienia tylko formę zapisu danych według publicznego algorytmu (odwracalne dla każdego). Szyfrowanie ukrywa treść tak, że odczytać ją może tylko ktoś, kto zna klucz.

### Pytania kontrolne
1. **Jaki problem rozwiązuje Base64?** Pozwala przesłać dowolne dane binarne kanałem, który obsługuje tylko tekst (e-mail, JSON, URL, nagłówki HTTP).
2. **Dlaczego zmiana wyglądu danych nie oznacza ich ochrony?** Bo algorytm jest publiczny i nie używa sekretu — każdy może wykonać operację odwrotną.
3. **Czy osoba mająca tylko secret.base64.txt odzyska treść?** Tak, wystarczy `base64 -d`.
4. **Kiedy Base64 jest użyteczne?** Przy załącznikach e-mail, osadzaniu obrazków w HTML/CSS (data URI), przesyłaniu kluczy/certyfikatów w PEM, przekazywaniu danych binarnych w JSON.

---

## 2. Hash pliku i integralność

1. Oblicz skrót SHA-256 pliku **secret.txt**.

sha256sum secret.txt
Wynik: 18a64d61a7b3f1735b4e0d148cb85a4d5e8e5b7a23acc1f9a896555450e5101a  secret.txt

2. Zapisz pierwszy wynik w pliku **secret.sha256**.

sha256sum secret.txt > secret.sha256
```

3. Zanotuj wartość skrótu również w **lab-notes.md**.

echo "Pierwszy SHA-256: $(cut -d' ' -f1 secret.sha256)" >> lab-notes.md
```

4. Dodaj do **secret.txt** krótką linię testową.

echo "linia testowa" >> secret.txt
```

5. Ponownie oblicz skrót SHA-256 zmodyfikowanego pliku.

sha256sum secret.txt
```

6. Porównaj nową wartość z wartością zapisaną wcześniej.

sha256sum -c secret.sha256
```
Wynik: `secret.txt: FAILED` — skrót jest zupełnie inny.

7. Ustal, czy na podstawie samego skrótu można odtworzyć treść pliku.
Nie — funkcja skrótu jest jednokierunkowa, z 64 znaków hex nie da się odtworzyć dowolnie długiej treści.

8. Przywróć **secret.txt** z pliku startowego znajdującego się w **materials**.

cp ../materials/sample-secret.txt secret.txt
```

9. Ponownie oblicz skrót i sprawdź, czy odpowiada wynikowi sprzed modyfikacji.

sha256sum -c secret.sha256
```
Wynik: `secret.txt: OK`

### Notatki
- Pierwszy skrót: `‹wartość›`
- Skrót po zmianie: `‹wartość›`
- Dodanie jednej linii zmieniło cały skrót, nie tylko jego fragment (efekt lawinowy).
- Hash nie zapewnia poufności, bo nie ukrywa danych — tylko tworzy ich „odcisk palca”. Służy do sprawdzania integralności.
- Funkcja skrótu nie jest odwracalna.

### Pytania kontrolne
1. **Co oznacza zgodność dwóch skrótów?** Że pliki są (z praktyczną pewnością) identyczne — kolizja SHA-256 jest w praktyce niemożliwa do znalezienia.
2. **Czy różne skróty dowodzą innej zawartości?** Tak — ta sama treść zawsze daje ten sam skrót, więc różny skrót oznacza różną treść.
3. **Dlaczego hash nie jest szyfrowaniem?** Szyfrowanie jest odwracalne kluczem, a hash nie da się odwrócić i nie używa klucza.

---

## 3. Szyfrowanie symetryczne AES

1–5. Zaszyfruj **secret.txt** algorytmem AES-256-CBC z losową solą i PBKDF2, zapisz jako **secret.enc**.

openssl enc -aes-256-cbc -salt -pbkdf2 -iter 100000 -in secret.txt -out secret.enc
```
- `enc` – szyfrowanie symetryczne,
- `-aes-256-cbc` – algorytm AES, klucz 256 bitów, tryb CBC,
- `-salt` – losowa sól dodawana do hasła,
- `-pbkdf2 -iter 100000` – wyprowadzenie klucza z hasła funkcją PBKDF2 (100 000 iteracji),
- `-in` / `-out` – plik wejściowy i wyjściowy.

OpenSSL pyta o hasło — podaję hasło testowe używane tylko w tym laboratorium.

6. Porównaj typ, rozmiar i wygląd **secret.txt** oraz **secret.enc**.

file secret.txt secret.enc
ls -l secret.txt secret.enc
```

7. Spróbuj wyświetlić zawartość szyfrogramu i zapisz obserwację.

xxd secret.enc | head
```
Widać nagłówek `Salted__`, a dalej losowo wyglądające bajty. Szyfrogram jest trochę większy od oryginału (sól + dopełnienie bloku).

8. Odszyfruj **secret.enc** poprawnym hasłem do pliku **decoded.txt**.

openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 -in secret.enc -out decoded.txt
```
(`-d` – deszyfrowanie, parametry muszą być takie same jak przy szyfrowaniu)

9. Porównaj **decoded.txt** z **secret.txt** w sposób pozwalający wykryć każdą różnicę.

cmp secret.txt decoded.txt && echo "Pliki identyczne"
```

10–12. Spróbuj odszyfrować błędnym hasłem do osobnego pliku i zachowaj komunikat błędu.

openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 -in secret.enc -out wrong-password.txt 2> wrong-password.err
echo $?
cat wrong-password.err
```
Kod wyjścia `1`, komunikat `bad decrypt`. Plik **decoded.txt** nie został nadpisany, bo wynik idzie do **wrong-password.txt**.

### Notatki
- Algorytm i tryb: AES-256 w trybie CBC.
- Sól: losowa wartość dołączana do hasła, dzięki której to samo hasło daje za każdym razem inny klucz i inny szyfrogram (utrudnia ataki tablicami tęczowymi).
- PBKDF2: zamienia hasło w klucz, wielokrotnie powtarzając obliczenia — spowalnia zgadywanie haseł metodą brute force.
- Porównanie `cmp`: pliki identyczne.
- Błędne hasło: OpenSSL zgłasza `bad decrypt` i zwraca kod 1; plik wyjściowy jest pusty albo zawiera śmieci.
- Nieczytelny wygląd nie dowodzi poprawnego szyfrowania — Base64 też wygląda na nieczytelny. O bezpieczeństwie decyduje algorytm, klucz i poprawne parametry, a nie wygląd danych.

### Pytania kontrolne
1. **Dlaczego potrzebny jest ten sam sekret?** W szyfrowaniu symetrycznym ten sam klucz służy do szyfrowania i deszyfrowania.
2. **Jaką rolę pełni sól?** Sprawia, że to samo hasło daje różne klucze i szyfrogramy, przez co nie da się użyć gotowych tablic hashy.
3. **Dlaczego hasło człowieka nie jest dobrym kluczem?** Jest krótkie, przewidywalne i ma małą entropię. Trzeba je wzmocnić funkcją KDF (PBKDF2), żeby powstał klucz odpowiedniej długości.
4. **Co może się stać przy błędnym haśle?** Zwykle błąd `bad decrypt` (niepoprawne dopełnienie). Czasem dopełnienie przypadkiem się zgadza i powstaje plik ze śmieciami — dlatego trzeba sprawdzać wynik.

---

## 4. Hasło, klucz i losowość

1–2. Wygeneruj kryptograficznie losowy sekret 32 bajtów i zapisz go w hex jako **aes.key**.

openssl rand -hex 32 > aes.key
```

3. Sprawdź liczbę znaków w pliku.

wc -c aes.key
```
Wynik: `65` (64 znaki hex + znak nowej linii).

4. Zależność 32 bajtów, 256 bitów i 64 znaków hex.
1 bajt = 8 bitów, więc 32 B × 8 = 256 bitów. Jeden znak hex to 4 bity, więc 1 bajt = 2 znaki hex → 32 × 2 = 64 znaki.

5. Ustal, czy narzędzie dodało znak końca linii.

tail -c 1 aes.key | xxd
00000000: 0a
Wynik `0a` — tak, na końcu jest znak nowej linii (`\n`).

6–7. Zaszyfruj **secret.txt**, pobierając sekret z **aes.key**, i zapisz jako **secret-with-keyfile.enc**.

openssl enc -aes-256-cbc -salt -pbkdf2 -kfile aes.key -in secret.txt -out secret-with-keyfile.enc
```
(`-kfile` – sekret jest czytany z pierwszej linii pliku zamiast z klawiatury)

8. Odszyfruj go do **decoded-with-keyfile.txt**.

openssl enc -d -aes-256-cbc -pbkdf2 -kfile aes.key -in secret-with-keyfile.enc -out decoded-with-keyfile.txt
```

9. Porównaj wynik z pierwotnym plikiem.

cmp secret.txt decoded-with-keyfile.txt && echo "Pliki identyczne"
Wynik: Pliki identyczne

10. Ogranicz uprawnienia **aes.key** tylko do właściciela.

chmod 600 aes.key
```

11. Sprawdź i zanotuj końcowe uprawnienia.

ls -l aes.key
stat -c '%a %A %U' aes.key
Wynik: 600 -rw------- kali
Wynik: `600 -rw-------` — tylko właściciel może czytać i zapisywać.

12. Co się stanie, jeśli napastnik zdobędzie szyfrogram i **aes.key**?
Poufność przepada całkowicie — może odszyfrować dane tą samą komendą co ja.

### Notatki
- Losowość: `openssl rand` (kryptograficznie bezpieczny generator liczb losowych).
- Długość: 32 bajty = 256 bitów = 64 znaki hex (+ `\n`).
- Hasło człowieka jest krótkie i przewidywalne. Losowy sekret ma pełne 256 bitów entropii i nie da się go odgadnąć.
- Uprawnienia: `600 (-rw-------)`.
- Ograniczenia: uprawnienia nie chronią przed rootem, przed kimś z fizycznym dostępem do dysku, przed kopią zapasową ani przed przejęciem mojego konta.

### Pytania kontrolne
1. **Dlaczego losowy sekret jest trudniejszy do odgadnięcia?** Ma 2^256 możliwych wartości i żadnego wzorca, a hasła ludzi opierają się na słowach i datach.
2. **Czy uprawnienia szyfrują plik?** Nie — plik dalej leży na dysku jawnym tekstem, uprawnienia tylko ograniczają dostęp w systemie.
3. **Gdzie przechowywać klucz w produkcji?** W menedżerze sekretów lub KMS (np. HashiCorp Vault, AWS KMS, Azure Key Vault), w HSM albo TPM — oddzielnie od danych.
4. **Jak utrata klucza wpływa na dane?** Bez klucza danych nie da się odzyskać — są w praktyce utracone.
5. **Dlaczego klucz obok szyfrogramu osłabia ochronę?** Kto ukradnie jedno, ukradnie też drugie — to jak zostawić klucz w zamku.

---

## 5. Generowanie pary kluczy RSA

1–3. Wygeneruj prywatny klucz RSA 3072 bitów jako **private-rsa.pem** i od razu ogranicz uprawnienia.

(umask 077; openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:3072 -out private-rsa.pem)
chmod 600 private-rsa.pem
```
(`umask 077` sprawia, że plik od początku powstaje z uprawnieniami tylko dla właściciela)

4–5. Wyprowadź klucz publiczny i zapisz jako **public-rsa.pem**.

openssl pkey -in private-rsa.pem -pubout -out public-rsa.pem
```

Sprawdzenie, że klucz publiczny pochodzi z tego klucza prywatnego:

diff <(openssl pkey -in private-rsa.pem -pubout) public-rsa.pem && echo "Para kluczy pasuje"
```

6. Porównaj nagłówki, rozmiary i uprawnienia obu plików.

head -1 private-rsa.pem public-rsa.pem
ls -l private-rsa.pem public-rsa.pem
openssl pkey -in private-rsa.pem -text -noout | head -1
```
Wynik:
==> private-rsa.pem <==
-----BEGIN PRIVATE KEY-----
==> public-rsa.pem <==
-----BEGIN PUBLIC KEY-----
-rw------- 1 kali kali 2488 Oct  7 11:37 private-rsa.pem
-rw-rw-r-- 1 kali kali  625 Oct  7 11:37 public-rsa.pem
Private-Key: (3072 bit, 2 primes)


7. Który plik można udostępnić?
Tylko **public-rsa.pem**.

8. Dlaczego klucz publiczny trzeba przekazać w sposób potwierdzający autentyczność?
Napastnik mógłby podmienić go na swój klucz (atak man-in-the-middle). Odbiorca szyfrowałby wtedy dane dla napastnika. Dlatego sprawdza się odcisk klucza albo używa certyfikatu.

openssl pkey -pubin -in public-rsa.pem -outform DER | sha256sum
```

### Notatki
- Pliki: `private-rsa.pem` (prywatny), `public-rsa.pem` (publiczny).
- Rozmiar: RSA 3072 bitów.
- Uprawnienia klucza prywatnego: `600`.
- Udostępnić można tylko klucz publiczny.
- Autentyczność ma znaczenie, bo podmieniony klucz publiczny pozwala napastnikowi czytać wiadomości przeznaczone dla mnie.

### Pytania kontrolne
1. **Dlaczego klucz publiczny można udostępnić?** Służy tylko do szyfrowania i weryfikacji podpisu — nie pozwala odszyfrować danych.
2. **Dlaczego klucz prywatny musi być tajny?** Pozwala odszyfrować wszystkie wiadomości i podpisywać w moim imieniu.
3. **Czy z klucza publicznego da się wyprowadzić prywatny?** Nie w praktyce — wymagałoby to rozłożenia ogromnej liczby na czynniki pierwsze.
4. **Co jeśli odbiorca zaakceptuje podmieniony klucz publiczny?** Napastnik odczyta wiadomości, może je zmienić i przesłać dalej (MITM).
5. **Poufność vs autentyczność klucza:** poufność to ukrycie klucza przed innymi (dotyczy prywatnego), autentyczność to pewność, że klucz naprawdę należy do danej osoby (dotyczy publicznego).

---

## 6. Szyfrowanie i deszyfrowanie małego pliku RSA

1. Utwórz krótki plik **short-message.txt**.

echo "Testowa wiadomość dla odbiorcy" > short-message.txt
wc -c short-message.txt
```

2–3. Zaszyfruj wiadomość kluczem publicznym i zapisz jako **short-message.rsa.enc**.

openssl pkeyutl -encrypt -pubin -inkey public-rsa.pem -in short-message.txt -out short-message.rsa.enc
 -pkeyopt rsa_padding_mode:oaep -in short-message.txt -out short-message.rsa.enc
```
- `pkeyutl -encrypt` – szyfrowanie kluczem asymetrycznym,
- `-pubin -inkey public-rsa.pem` – używamy klucza publicznego,
- `-pkeyopt rsa_padding_mode:oaep` – bezpieczne dopełnienie OAEP.

4–5. Odszyfruj kluczem prywatnym do **short-message.decoded.txt**.

openssl pkeyutl -decrypt -inkey private-rsa.pem -in short-message.rsa.enc -out short-message.decoded.txt
```

6. Porównaj wynik z oryginalną wiadomością.

cmp short-message.txt short-message.decoded.txt && echo "Pliki identyczne"
```

7. Dlaczego szyfrujemy publicznym, a odszyfrowujemy prywatnym?
Każdy może zaszyfrować wiadomość dla odbiorcy jego kluczem publicznym, ale odczytać ją może tylko on, bo tylko on ma klucz prywatny.

8–9. Próba zaszyfrowania dużego pliku.

head -c 1000 /dev/urandom > big.bin
openssl pkeyutl -encrypt -pubin -inkey public-rsa.pem -pkeyopt rsa_padding_mode:oaep -in big.bin -out big.rsa.enc 2> rsa-size-error.txt
cat rsa-size-error.txt
```
Komunikat: `data too large for key size`.

### Notatki
- Rozmiar wiadomości: `‹wynik wc -c›` bajtów.
- Porównanie: pliki identyczne.
- Klucz publiczny służy do szyfrowania, klucz prywatny do odszyfrowania.
- RSA 3072 z OAEP przyjmuje maksymalnie ok. 342 bajty (384 bajty klucza − 42 bajty dopełnienia). Plik 1000 B odrzucony: `data too large for key size`.

### Pytania kontrolne
1. **Dlaczego autor nie potrzebuje klucza prywatnego odbiorcy?** Do szyfrowania wystarczy klucz publiczny odbiorcy.
2. **Co umożliwiłoby ujawnienie klucza prywatnego?** Odszyfrowanie wszystkich wiadomości (także przechwyconych wcześniej) i podszywanie się pod właściciela.
3. **Dlaczego RSA ma limit rozmiaru?** Dane są traktowane jako jedna liczba, która musi być mniejsza od modułu klucza, a dopełnienie zajmuje dodatkowe bajty.
4. **Dlaczego nie szyfruje się dużego pliku blok po bloku RSA?** RSA jest bardzo wolne, szyfrogram rośnie, a szyfrowanie blokami bez trybu pracy nie chroni struktury danych.
5. **Jak rozwiązuje to szyfrowanie hybrydowe?** Dane szyfruje szybki AES, a RSA szyfruje tylko mały klucz AES.

---

## 7. Prosty model szyfrowania hybrydowego

1–3. Wygeneruj nowy 32-bajtowy sekret jako **hybrid-aes.key** i ogranicz dostęp.

(umask 077; openssl rand -hex 32 > hybrid-aes.key)
chmod 600 hybrid-aes.key
```

4–5. Zaszyfruj **secret.txt** tym sekretem do **hybrid-message.enc**.

openssl enc -aes-256-cbc -salt -pbkdf2 -kfile hybrid-aes.key -in secret.txt -out hybrid-message.enc
```

6–7. Zabezpiecz klucz symetryczny kluczem publicznym RSA odbiorcy jako **hybrid-aes.key.enc**.

openssl pkeyutl -encrypt -pubin -inkey public-rsa.pem -pkeyopt rsa_padding_mode:oaep -in hybrid-aes.key -out hybrid-aes.key.enc
```

8–10. Jako odbiorca odzyskaj klucz kluczem prywatnym jako **hybrid-aes.recovered.key** i ogranicz dostęp.

(umask 077; openssl pkeyutl -decrypt -inkey private-rsa.pem -pkeyopt rsa_padding_mode:oaep -in hybrid-aes.key.enc -out hybrid-aes.recovered.key)
chmod 600 hybrid-aes.recovered.key
cmp hybrid-aes.key hybrid-aes.recovered.key && echo "Klucz odzyskany poprawnie"
```
Wynik: Klucz odzyskany poprawnie

11–12. Odszyfruj **hybrid-message.enc** odzyskanym kluczem do **hybrid-message.decoded.txt**.

openssl enc -d -aes-256-cbc -pbkdf2 -kfile hybrid-aes.recovered.key -in hybrid-message.enc -out hybrid-message.decoded.txt
```

13. Porównaj wynik z **secret.txt**.

cmp secret.txt hybrid-message.decoded.txt && echo "Pliki identyczne"
```
Wynik: Pliki identyczne

14. Przepływ danych:
```
NADAWCA                                              ODBIORCA
secret.txt ──AES-256 (hybrid-aes.key)──► hybrid-message.enc ──AES-256 (recovered.key)──► hybrid-message.decoded.txt
                       ▲   (SYMETRYCZNIE)                                     ▲
hybrid-aes.key ──RSA (public-rsa.pem)──► hybrid-aes.key.enc ──RSA (private-rsa.pem)──► hybrid-aes.recovered.key
                     (ASYMETRYCZNIE)
```

### Notatki
- Zaszyfrowane dane: **hybrid-message.enc** (AES).
- Zaszyfrowany klucz: **hybrid-aes.key.enc** (RSA).
- Klucze RSA: szyfrowanie klucza AES → `public-rsa.pem` (nadawca), odzyskanie klucza AES → `private-rsa.pem` (odbiorca).
- Porównanie: **hybrid-message.decoded.txt** identyczny z **secret.txt**.
- Korzyść: AES szybko szyfruje dane dowolnej wielkości, a RSA rozwiązuje problem bezpiecznego przekazania klucza bez wcześniejszego wspólnego sekretu. Tak działa m.in. TLS i PGP.

---

## 8. CyberChef — kodowanie i szyfrowanie

Strona: **https://gchq.github.io/CyberChef/** (oficjalna instancja GCHQ). Do pola *Input* wklejam tylko testową treść `secret.txt`.

| Krok | Operacja w CyberChef | Odpowiednik w terminalu |
|---|---|---|
| 3. Base64 | **To Base64** | `base64 secret.txt` |
| 4. Odwrócenie | **To Base64** → **From Base64** | `base64 secret.txt \| base64 -d` |
| 6. Hex | **To Hex** | `xxd -p secret.txt` |
| 7. Powrót z hex | **To Hex** → **From Hex** | `xxd -p secret.txt \| xxd -r -p` |
| 8. URL encoding | **URL Encode** / **URL Decode** | `python3 -c "import urllib.parse;print(urllib.parse.quote('Ala ma kota & 100%?'))"` |
| 9. AES | **AES Encrypt** (Key, IV, Mode: CBC) | `openssl enc -aes-256-cbc ...` |
| 10. Deszyfrowanie | **AES Encrypt** → **AES Decrypt** (ten sam Key i IV) | `openssl enc -d -aes-256-cbc ...` |

Przykładowy przepis AES (test):
- **AES Encrypt**: Key = `00112233445566778899aabbccddeeff00112233445566778899aabbccddeeff` (HEX), IV = `000102030405060708090a0b0c0d0e0f` (HEX), Mode = CBC, Output = Hex
- **AES Decrypt**: ten sam Key i IV, Mode = CBC, Input = Hex

12. Przepis zapisuję przyciskiem **Save recipe** (np. jako `From_Base64()` / `AES_Encrypt(...)`).

### Notatki
- Kodowanie: Base64, Hex, URL encoding.
- Szyfrowanie: AES.
- Sekretu wymagał tylko AES (klucz + IV). Base64, Hex i URL encoding działają bez sekretu.
- Kodowanie odwraca każdy, kto zna nazwę operacji. Deszyfrowanie AES działa tylko z właściwym kluczem — przy złym kluczu wychodzą śmieci albo błąd.
- URL encoding zamienia znaki specjalne (spacja, `&`, `?`, `%`) na `%XX`, żeby można je było bezpiecznie przekazać w adresie URL.

---

## 9. Diagnostyka HTTPS

1. Wybieram domenę testową:

DOMENA=github.com
```

2–3. Połączenie TLS z portem 443 z podaniem nazwy serwera (SNI).

openssl s_client -connect $DOMENA:443 -servername $DOMENA -showcerts < /dev/null
```
- `-connect host:443` – połączenie z portem HTTPS,
- `-servername` – SNI, nazwa serwera wysłana podczas zestawiania połączenia,
- `-showcerts` – pokazuje cały łańcuch certyfikatów,
- `< /dev/null` – od razu kończy połączenie (punkt 9).

4. Łańcuch certyfikatów:

openssl s_client -connect $DOMENA:443 -servername $DOMENA < /dev/null 2>/dev/null | sed -n '/Certificate chain/,/---/p'
```
Wynik:
Certificate chain
 0 s:CN=github.com
   i:C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication CA DV E36
   a:PKEY: EC, (prime256v1); sigalg: ecdsa-with-SHA256
   v:NotBefore: Sep  1 00:00:00 2026 GMT; NotAfter: Nov 29 23:59:59 2026 GMT
 1 s:C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication CA DV E36
   i:C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication Root E46
   a:PKEY: EC, (prime256v1); sigalg: ecdsa-with-SHA384
   v:NotBefore: Mar 22 00:00:00 2021 GMT; NotAfter: Mar 21 23:59:59 2036 GMT
 2 s:C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication Root E46
   i:C=US, ST=New Jersey, L=Jersey City, O=The USERTRUST Network, CN=USERTrust ECC Certification Authority
   a:PKEY: EC, (secp384r1); sigalg: ecdsa-with-SHA384
   v:NotBefore: Mar 22 00:00:00 2021 GMT; NotAfter: Jan 18 23:59:59 2038 GMT

5–6. Podmiot, wystawca, nazwy domen i daty ważności certyfikatu serwera.

openssl s_client -connect $DOMENA:443 -servername $DOMENA < /dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```
Wynik:
subject=CN=github.com
issuer=C=GB, O=Sectigo Limited, CN=Sectigo Public Server Authentication CA DV E36
notBefore=Sep  1 00:00:00 2026 GMT
notAfter=Nov 29 23:59:59 2026 GMT
X509v3 Subject Alternative Name: 
    DNS:github.com, DNS:www.github.com

7. Wynegocjowana wersja TLS i zestaw szyfrów.

openssl s_client -connect $DOMENA:443 -servername $DOMENA < /dev/null 2>/dev/null | grep -E "Protocol|Cipher"
```
Wynik:
New, TLSv1.3, Cipher is TLS_AES_128_GCM_SHA256
Protocol: TLSv1.3

8. Wynik walidacji certyfikatu.

openssl s_client -connect $DOMENA:443 -servername $DOMENA -verify_return_error < /dev/null 2>/dev/null | grep "Verify return code"
```
Oczekiwany wynik: `Verify return code: 0 (ok)`

9. Zakończenie połączenia — `< /dev/null` (albo w trybie interaktywnym `Ctrl+C` / wpisanie `Q`).

10. Same nagłówki odpowiedzi przez HTTPS.

curl -sI https://$DOMENA
```

11. Analogiczna próba przez HTTP.

curl -sI http://$DOMENA
```

12. Czy HTTP przekierowuje na HTTPS?

curl -sI http://$DOMENA | grep -iE "^HTTP|^location"
curl -sIL http://$DOMENA | grep -iE "^HTTP|^location"
```
Kod `301` z nagłówkiem `Location: https://...` oznacza przekierowanie na HTTPS.

Dodatkowo nagłówek HSTS:

curl -sI https://$DOMENA | grep -i strict-transport-security
Wynik: strict-transport-security: max-age=31536000; includeSubdomains; preload

13. Ochrona kanału a bezpieczeństwo aplikacji — patrz wnioski poniżej.

### Notatki
- Domena: `‹github.com›`
- Podmiot certyfikatu: `‹subject=CN=...›`
- Wystawca: `‹issuer=...›`
- Ważność: `‹notBefore ... / notAfter ...›`
- Wersja TLS: `‹np. TLSv1.3›`
- Szyfr: `‹np. TLS_AES_128_GCM_SHA256›`
- Walidacja: `‹Verify return code: 0 (ok)›`
- HTTP: `‹301 → Location: https://...›`
- Wniosek: HTTPS szyfruje i uwierzytelnia transmisję między przeglądarką a serwerem, ale nie chroni przed błędami samej aplikacji.

### Pytania kontrolne
1. **Dlaczego klient sprawdza nazwę domeny w certyfikacie?** Żeby upewnić się, że rozmawia z właściwym serwerem, a nie z kimś, kto przedstawia ważny certyfikat innej domeny.
2. **Jaką rolę pełni urząd certyfikacji?** CA potwierdza, że klucz publiczny należy do właściciela domeny, i podpisuje certyfikat. Przeglądarka ufa CA z wbudowanej listy.
3. **Co oznacza data wygaśnięcia?** Po tej dacie certyfikat jest nieważny i przeglądarka pokaże ostrzeżenie. Ograniczony czas życia zmniejsza skutki wycieku klucza.
4. **Dlaczego SNI ma znaczenie?** Na jednym adresie IP może działać wiele domen — SNI mówi serwerowi, który certyfikat ma przedstawić. Bez niego można dostać zły certyfikat.
5. **Co chroni HTTPS?** Poufność (szyfrowanie), integralność (brak modyfikacji w trakcie przesyłania) i autentyczność serwera (certyfikat).
6. **Czego HTTPS nie naprawia?** SQL injection, XSS, CSRF, błędów logiki i autoryzacji, słabych haseł, wycieków danych z serwera, podatnych bibliotek — HTTPS chroni tylko drogę danych, nie samą aplikację.
