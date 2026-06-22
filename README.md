# Dokumentacja Projektu - Podstawy Programowania

Zestawienie rozwiązań zadań z przedmiotu Podstawy programowania. Dokument zawiera polecenia systemu Linux, skrypty powłoki Bash oraz instrukcje dla narzędzi kolekcji GNU i ImageMagick zrealizowane w środowisku MSYS2.

---

## Zależności i Środowisko Pracy

Wszystkie operacje zrealizowano w terminalu MSYS2 (architektura MINGW64 / UCRT64). Skrypty wymagają następujących pakietów:

* **Powłoka systemowa:** Bash
* **Narzędzia kolekcji GNU:** `cat`, `grep`, `sed`, `awk`, `diff`, `patch`, `paste`, `column`, `tr`
* **Zarządzanie wersjami i integralność:** `git`, `md5sum`
* **Archiwizacja danych:** `zip`, `unzip`, `p7zip` (polecenie `7z`)
* **Przetwarzanie obrazów:** `ImageMagick` (pakiet `mingw-w64-ucrt-x86_64-imagemagick`)
* **Konwersja znaków końca linii:** `dos2unix`

---

## Szczegółowy Opis Zadań

### Zadanie 1: Zespoły i narzędzia
* **Cel:** Inicjalizacja repozytorium Git oraz konfiguracja uprawnień dostępu dla członków grupy projektowej.

### Zadanie 2: MSYS2 i konfiguracja środowiska
* **Cel:** Instalacja środowiska MSYS2 oraz pobranie wymaganych pakietów narzędziowych za pomocą menedżera `pacman`.
* **Polecenie:**
    ```bash
    pacman -Syu
    pacman -S vim nano less diffutils zip unzip dos2unix patch mingw-w64-ucrt-x86_64-imagemagick
    ```

### Zadanie 3: Niesforne dane
* **Cel:** Reorganizacja jednowymiarowej listy danych z pliku `dane.txt` do ustrukturyzowanej tabeli trójkolumnowej.
* **Polecenie:**
    ```bash
    (sed -i '1i x\ny\nz' dane.txt; paste - - - < dane.txt) > wynik.txt | column -t
    ```
* Brak pewności — wymagana weryfikacja. Składnia polecenia oparta na fragmentarycznych danych z dostarczonego pliku PDF.

### Zadanie 4: Dodawanie poprawek
* **Cel:** Generowanie pliku różnicowego na podstawie dwóch wersji dokumentu tekstowego. Zastosowano konwersję formatu końca linii oraz walidację integralności algorytmem MD5.
* **Polecenie:**
    ```bash
    unix2dos lista-pop.txt
    unix2dos lista.txt
    diff lista.txt lista-pop.txt > lista.patch
    diff --brief lista.txt lista-pop.txt
    md5sum lista.txt lista-pop.txt
    ```

### Zadanie 5: Z CSV do SQL i z powrotem
* **Cel:** Ekstrakcja danych z formatu CSV do instrukcji wprowadzania danych `INSERT INTO` (SQL), a w drugim etapie transformacja powrotna skryptu SQL do pliku CSV z ucięciem wartości milisekund w stemplach czasu.
* **Polecenie:**
    ```bash
    awk -F: 'NR>1 {print "INSERT INTO stepsData (time, intensity, steps) VALUES ("$1", "$2", "$3 ");"}' steps-2sql.csv > import.sql
    awk 'BEGIN {print "dateTime,steps,synced"} /INSERT INTO/ {print substr($8, 1, length($8)-3)":"$10":"$12}' steps-2csv.sql > sql_na_excela.csv
    ```
* Brak pewności — wymagana weryfikacja. Oryginalny kod w dokumentacji PDF jest częściowo nieczytelny. Powyższa struktura stanowi rekonstrukcję.

### Zadanie 6: Marudny tłumacz
* **Cel:** Modyfikacja plików konfiguracyjnych JSON. Powielenie linii z kluczami (z zastosowaniem komentarza dla linii bazowych) oraz wyodrębnienie wyłącznie nowych fraz względem starszej wersji pliku.
* **Polecenie:**
    ```bash
    awk NF en-7.2.json5 | awk '{print "//" $0; print; print ""}' | sed '1d;$d' | sed '$d' | sed '$d' | awk '{print} END {print "}"}' > pl-7.2.json5
    awk NF en-7.4.json5 | awk '{print "//" $0; print; print ""}' | sed '1d;$d' | sed '$d' | sed '$d' | awk '{print} END {print "}"}' > pl-7.4.json5
    ```

### Zadanie 7: Fotografik gamoń
* **Cel:** Wypakowanie zawartości archiwów ZIP oraz wsadowa konwersja plików graficznych do formatu JPG. Wymuszono parametry wyjściowe: rozdzielczość 96 DPI i wysokość 720px.
* **Polecenie:**
    ```bash
    mkdir tmp && cd tmp
    pacman -S unzip zip --noconfirm
    unzip -q ../kopie-1.zip && unzip -q ../kopie-2.zip
    for f in *.zip; do unzip -q -o "$f"; rm "$f"; done
    pacman -S mingw-w64-x86_64-imagemagick --noconfirm
    /mingw64/bin/mogrify -format jpg -resize x720 -density 96 -units PixelsPerInch *.png
    find . -type f ! -name "*.jpg" -delete
    zip -q ../gotowe_zdjecia.zip *.jpg
    cd .. && rm -rf tmp
    ```

### Zadanie 8: Wszędzie te PDF-y
* **Cel:** Agregacja przeskalowanych obrazów JPG do pojedynczego dokumentu PDF w formacie A4. Zastosowano układ kafelkowy 2x4 z naniesieniem nazw plików jako etykiet tekstowych.
* **Polecenie:**
    ```bash
    unzip -q gotowe_zdjecia.zip
    /mingw64/bin/montage -label "%f" -geometry +10+20 -tile 2x4 -page A4 *.jpg portfolio.pdf
    rm *.jpg
    ```

### Zadanie 9: Porządki w kopiach zapasowych
* **Cel:** Automatyczna segregacja archiwów. Przekształcenie płaskiej struktury plików ZIP do hierarchicznego drzewa podkatalogów (podział na rok i miesiąc) bazując na nazewnictwie plików.
* **Skrypt iteracyjny:**
    ```bash
    mkdir kopie && cd kopie
    unzip ../kopie-1.zip && unzip -q ../kopie-2.zip
    for plik in *.zip; do rok=$(echo "$plik" | cut -d- -f1); miesiac=$(echo "$plik" | cut -d- -f2); mkdir -p "$rok/$miesiac"; mv "$plik" "$rok/$miesiac/"; done
    ```

### Zadanie 10: Galeria dla grafika
* **Cel:** Skrypt wykonujący iterację po nazwach plików w katalogu celem wygenerowania powtarzalnych bloków kodu HTML, stanowiących wsad do szablonu galerii internetowej.
* **Skrypt bash:**
    ```bash
    #!/bin/bash
    for plik in *.jpg; do
        echo '<div class="responsive">'
        echo '  <div class="gallery">'
        echo '    <a target="_blank" href="'"$plik"'">'
        echo '      <img src="'"$plik"'">'
        echo '    </a>'
        echo '    <div class="desc">'"$plik"'</div>'
        echo '  </div>'
        echo '</div>'
    done > wstawka.html
    ```
