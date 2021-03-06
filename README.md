# **MySQL** dla opornych - Powtórka
## 1. Podstawy.

Rodzaje znaków:
```diff
+ `...` (Backtick)  - Używany do nazw struktur: Baz danych, tabel, kolumn.
- '...' (Apostof)   - Używany do ciągów np: Wartości pól, loginy, hasła.
```

Logowanie do bazy danych:
```
mysql -u root
mysql -u root -p
```
Wersja bazy oraz aktualnie zalogowany użytkownik:
```
\s
```
Wylistowanie baz danych:
```sql
SHOW DATABASES;
```
Wybieranie konkretnej bazy danych:
```sql
USE nazwa_bazy;
```
- Wyświetlenie aktualnie wybranej bazy: ```SELECT DATABASE();```

Wylistowanie wszystkich tabel:
```sql
SHOW TABLES;
```
Wyświetlanie struktury konkretnej tabeli:
```sql
DESCRIBE nazwa_tabeli;
```
Wylistowanie wszystkich użytkowników:
```sql
SELECT User, Host FROM mysql.user;
```
***
## 2. **CREATE** - Tworzenie struktur.
> Składnia: 
<b><span style="font-family: Consolas">CREATE <i>[typ] [nazwa]</i>;</span></b>

### Tworzenie użytkownika:
```sql
CREATE USER 'uzytkownik'@'localhost' IDENTIFIED BY 'haslo';
```
### Tworzenie bazy danych:
```sql
CREATE DATABASE nazwa_bazy;
CREATE DATABASE nazwa_bazy DEFAULT CHARACTER
SET utf8 COLLATE utf8_unicode_ci;
```
### Tworzenie tabel:
> Składnia:
```sql
CREATE TABLE nazwa_tabeli(
    n_kolumny_1 [typ] [atrybuty],
    n_kolumny_2 [typ] [atrybuty]
);
```

Typy danych:

| Typ             | Opis                                |
|-----------------|-------------------------------------|
| TINYINT         | Małe liczby całkowite do 255.       |
| SMALLINT        | Średnie liczby całkowite do 32k.    |
| <u>INT</u>             | Liczby całkowite ponad 2 miliardy.  |
| BOOL            | Wartość true/false.                 |
| FLOAT(precyzja) | Liczby zmiennoprzecinkowe.          |
| CHAR(x)         | Pole tekstowe o długości = x        |
| <u>VARCHAR(x)</u>      | Pole tekstowe o długości =< x       |
| TEXT            | Pole tekstowe o nieznanej długości. |
| DATETIME        | Data oraz godzina.                  |

Atrybuty:

| Atrybut        | Opis                                      |
|----------------|-------------------------------------------|
| NOT NULL       | Rekord nie może być puste.                |
| AUTO_INCREMENT | Rekord będzie automatycznie numerowany.   |
| <u>PRIMARY KEY</u>    | Pole będzie kluczem głównym w tabeli.     |
| FOREIGN KEY    | Pole będzie kluczem obcym w tabeli.       |
| UNIQUE         | Rekord będzie unikalny.                   |
| DEFAULT '...'  | Domyślna wartość rekordu ustawiona na ... |
| CHECK          | Warunek dla wprowadzanych rekordów.       |
| UNSIGNED       | Wartość dodatnia.                         |

Tworzenie przykładowej tabeli:
```sql
CREATE TABLE obiady
(
    obiad_id    INT         PRIMARY KEY AUTO_INCREMENT,
    nazwa       VARCHAR(30) NOT NULL UNIQUE,
    jadalny     BOOL        DEFAULT TRUE,
    cena        FLOAT(6,2)  CHECK(cena>=1)
);
---
CREATE TABLE zamowienia
(
    zamowienie_id   INT         AUTO_INCREMENT,
    data_zamowienia DATETIME    NOT NULL,
    obiad_id        INT         NOT NULL,
    ilosc           SMALLINT    UNSIGNED,
    PRIMARY KEY(`zamowienie_id`), -- Inny sposób definiowania pk
    FOREIGN KEY(`obiad_id`) REFERENCES obiady(`obiad_id`)
);
```
> Klucz główny z kilku kolumn: ```PRIMARY KEY(k1,k2)```

Zmiana nazwy tabeli
```sql
RENAME TABLE 'nazwa_tabeli' TO 'nowa_nazwa';
```
### Tworzenie indeksu:
> Składnia: 
<b><span style="font-family: Consolas">CREATE <u>UNIQUE</u> INDEX <i>[nazwa_indeksu]</i> ON <i>[nazwa_tabeli]</i>
(kolumny);</span></b>

* `UNIQUE` - Tworzy unikalny indeks - nie jest wymagane.

Przykładowy indeks:
```sql
CREATE UNIQUE INDEX i_nazwa ON `obiady` (`nazwa`);
CREATE INDEX i_zamowienie ON `zamowienia` (`data_zamowienia`,`obiad_id`); --Index z kilku kolumn.
```
Indeks przy tworzeniu tabeli:
```sql
CREATE TABLE dostawcy
(
    dostawca_id INT         PRIMARY KEY AUTO_INCREMENT,
    nazwa       VARCHAR(10) NOT NULL,
    telefon     VARCHAR(9)  NOT NULL DEFAULT '997',
    ...,
    INDEX i_dostawca(`dostawca_id`),
    INDEX i_nazwa_tel(`nazwa`,`telefon`),
);
```
***
## 3. **DROP** - Usuwanie struktur.
> Składnia: 
<b><span style="font-family: Consolas">DROP <i>[typ] [nazwa]</i></span>;</b>
### Usuwanie bazy:
```sql
DROP DATABASE nazwa_bazy;
```
### Usuwanie tabeli:
```sql
DROP TABLE nazwa_tabeli;
```
### Usuwanie indeksu:
```sql
DROP INDEX `nazwa_indeksu` ON `nazwa_tabeli`;
```
### Usuwanie użytkownika:
```sql
DROP USER 'nazwa'@'localhost';
```
***
## 4. **ALTER** - Modyfikowanie struktur.
> Składnia:
<b><span style="font-family: Consolas">ALTER <i>[obiekt] [nazwa_obiektu] [specyfikacja]</i></span>;</b>

### Dodawanie klucza głównego do tabeli:
> <b><span style="font-family: Consolas">ALTER TABLE [tabela] ADD CONSTRAINT [nazwa_klucza] PRIMARY KEY([kolumny])</span>;</b>
```sql
ALTER TABLE `uzytkownicy` ADD CONSTRAINT pk_uzytkownik PRIMARY KEY(id);
ALTER TABLE `uzytkownicy` ADD CONSTRAINT pk_uzytkownik PRIMARY KEY(id,pesel);
```
### Usuwanie klucza głównego z tabeli:
```sql
ALTER TABLE `uzytkownicy` DROP PRIMARY KEY;
```
### Dodawanie nowej kolumny:
> <b><span style="font-family: Consolas">ALTER TABLE [tabela] ADD COLUMN [nazwa_kolumny] [typ] <u>[atrybuty]</u></span>;</b>
```sql
ALTER TABLE `uzytkownicy` ADD COLUMN `uzytkownik_id` INT AUTO_INCREMENT PRIMARY KEY;
ALTER TABLE `uzytkownicy` ADD COLUMN `strona` VARCHAR(40) NOT NULL;
ALTER TABLE `uzytkownicy` ADD COLUMN `wiek` INT CHECK (wiek > 18) NOT NULL;
```
### Usunięcie istniejącej kolumny:
> <b><span style="font-family: Consolas">ALTER TABLE [tabela] DROP [kolumna]</span>;</b>
```sql
ALTER TABLE `uzytkownicy` DROP `wiek`;
```
### Zmiana nazwy kolumny:
> <b><span style="font-family: Consolas">ALTER TABLE [tabela] CHANGE COLUMN [nazwa_kolumny] [nowa_nazwa] [typ]</span>;</b>
```sql
ALTER TABLE `uzytkownicy` CHANGE COLUMN `uzytkownik_id` `u_id` INT;
```
* <b>Trzeba określić typ nowej kolumny.</b>
### Dodawanie/usuwanie atrybutów do kolumny:
> <b><span style="font-family: Consolas">ALTER TABLE [tabela] MODIFY [nazwa_kolumny] [typ] [atrybuty]</span>;</b>

> * Użycie `MODIFY` re-deklaruje typ i atrybuty kolumny, jeżeli chcemy dodać nowy atrybut musimy przepisać jego stare atrybuty oraz jego typ.
```sql
ALTER TABLE `uzytkownicy` MODIFY `strona` VARCHAR(40) DEFAULT 'http://google.com'; -- Stare atrybuty pola `strona` zostały usunięte - nadano nowe: DEFAULT...
ALTER TABLE `uzytkownicy` MODIFY `strona` VARCHAR(40) NOT NULL DEFAULT 'http://google.com';
ALTER TABLE `uzytkownicy` MODIFY `strona` VARCHAR(10);
```
### Dodawanie atrybutu UNIQUE:
```sql
ALTER TABLE `uzytkownicy` MODIFY `ip` VARCHAR(10) UNIQUE;
```
Albo:
```sql
ALTER TABLE `uzytkownicy` ADD CONSTRAINT unique_ip UNIQUE('ip');
```
### Usuwanie atrybutu UNIQUE:
> * Atrybutu <u>UNIQUE</u> oraz <u>PRIMARY KEY</u> nie da się usunąć poprzez ```MODIFY``` - <b>W tych przypadkach działa tylko na dodawanie</b>.

Usunięcie UNIQUE:
```sql
 ALTER TABLE `uzytkownicy` DROP INDEX `nazwa_kolumny`;
```
Usunięcie PRIMARY KEY:
```sql
 ALTER TABLE `uzytkownicy` DROP PRIMARY KEY;
```
***
## 5. **GRANT** - Nadawanie przywilejów.
> Składnia:
<b><span style="font-family: Consolas">GRANT <i>[przywileje]</i>
ON <i>[obiekt]</i>
TO <i>[uzytkownik] <u>IDENTIFIED BY `'haslo'`</u></i></span>;</b>

* <b>`przywileje`</b>  - Konkretne uprawnienia rozdzielone ```,``` albo:
    - <b>`ALL`</b> - Wszystkie uprawnienia.
    - <b>`USAGE`</b> - Brak uprawnień - pozwala na zalogowanie.
* <b>`obiekt`</b> - Struktura/y do których przypisujemy uprawnienia np:
    - `*.*` - Do wszystkich baz i wszystkich tabel tych baz.
    - `baza.*` - Wszystkie tabele bazy o nazwie `'baza'`.
    - `baza.kolumna` - Uprawnienia przypisane do kolumny `'kolumna'` w bazie `'baza'`.
* <b>`uzytkownik`</b> - Nazwa użytkownika któremu przypisujemy uprawnienia.
* <u>`IDENTIFIED BY '...'`</u> - Tworzy nowego użytkownika.

| Uprawnienie | Opis                                |
|-------------|-------------------------------------|
| SELECT      | Wyszukiwanie rekordów w tabelach .  |
| INSERT      | Wstawianie nowych wierszy do tabeli.|
| UPDATE      | Zmiana wartości zapisane w tabeli.  |
| DELETE      | Usuwanie wartości z tabeli.         |
| INDEX       | Tworzenie i usuwanie indeksów.      |
| ALTER       | Modyfikacja struktur.               |
| CREATE      | Tworzenie nowych struktur.          |
| DROP        | Usuwanie struktur.                  |
| GRANT       | Nadawanie uprawnień.                |

Przykład:
```sql
GRANT SELECT, INSERT, UPDATE
ON ksiegarnia_internetowa.*
TO 'klient'
IDENTIFIED BY 'haslo'; --Dodatkowo tworzymy nowego użytkownika
```
Nadawanie wszystkich<span style="color: red;"><b>*</b></span> uprawnień:
```sql
GRANT ALL
ON *.*
TO 'klient';
```
> <span style="color: red;"><b>*</b></span> - Nie zostało nadane uprawnienie `GRANT` - aby je nadać używamy:
```sql
GRANT ALL
ON *.*
TO 'klient' WITH GRANT OPTION;
```
Wylistowanie uprawnień użytkownika:
```sql
SHOW GRANTS FOR 'nazwa_uzytkownika'@'localhost';
```
## 5. **REVOKE** - Odbieranie przywilejów.
> Składnia:
<b><span style="font-family: Consolas">REVOKE <i>[przywileje]</i>
ON <i>[obiekt]</i>
FROM <i>[uzytkownik]</i>;</b>

Odbieranie konkretnych uprawnień do wszystkich tabel w `baza`:
```sql
REVOKE ALTER, CREATE, DROP
ON `baza`.*
FROM 'klient';
```
Odbieranie wszystkich uprawnień:
```sql
REVOKE ALL ON * FROM 'klient'@'localhost';
REVOKE GRANT OPTION ON * FROM 'klient'@'localhost';
```
***
## 6. **INSERT** - Wstawianie rekordów.
> Składnia:
```sql
INSERT INTO `nazwa_tabeli` (kolumna1,kolumna2, ...)
VALUES (wartosc1,wartosc2, ...);
```
* <b>`nazwa_tabeli`</b>  - Nazwa tabeli do której wstawiamy rekordy.
* <u>`(kolumna1, kolumna2)`</u> - Nazwy kolumn do których wstawiamy dane - <b>Nie jest wymagane</b>.
* <b>`(wartosc1,wartosc2)`</b> - Wartości jakie mają zostać przypisane do kolumn.

### Wstawianie danych do wszystkich kolumn:
> **Ważne:** Liczba wartości musi być równa ilości kolumn.
```sql
INSERT INTO `obiady`
VALUES (NULL, 'Big kebab baranina', TRUE, 15.0);
```
> Jeżeli kolumna nie posiada atrybutu <u>`NOT NULL`</u> możemy jako jej wartość wstawić <b>`NULL`</b>.

> Po wstawieniu <b>`NULL`</b> do kolumny z atrybutem <u>`AUTO_INCREMENT`</u> wartość zostanie zwiększona automatycznie. (Np. pole z kluczem głównym)

### Wstawianie danych do konkretnych kolumn:
```sql
INSERT INTO `obiady` (`nazwa`,`jadalny`)
VALUES ('Bigos', FALSE);
```
> W niewymienione pola automatycznie zostaje wstawiona wartość <b>`NULL`</b>.

### Dodawanie przez przypisanie:
```sql
INSERT INTO `obiady` SET nazwa='Kremówki', jadalny = TRUE, cena = 21.37;
```

### Wstawianie kilku rekordów jednocześnie:
```sql
INSERT INTO `obiady`
VALUES  (NULL, 'Hawajska', TRUE, 24.0),
        (NULL, 'Pierogi na szkolnej wigilii', FALSE, 14),
        (NULL, 'Pączki z biedry x12', FALSE, 4.50);
```
***
## 7. **UPDATE** - Modyfikacja danych.
> Składnia: 
<b><span style="font-family: Consolas">UPDATE [nazwa_tabeli] SET [kolumna1 = 'wartosc1', kolumna2 = 'wartosc2', ...]
<u>WHERE kolumna='warunek'</u></span>;</b>
* <u>`WHERE nazwa_kolumny='...'`</u>  - Modyfikacja wierszy które spełniają warunek określony po `WHERE`. Nie jest wymagany.

Przykład:
```sql
UPDATE `obiady` SET cena=9.97 WHERE nazwa='Hawajska';
UPDATE `obiady` SET nazwa='Bakłażan', jadalny=FALSE, cena=0.7 WHERE obiad_id = 1; 
```
Odniesienie do aktualnych wartości:
```sql
UPDATE `obiady` SET cena=cena + cena * 0.10;
```
***
## 8. **DELETE** - Usuwanie danych.
> Składnia: 
<b><span style="font-family: Consolas">DELETE FROM [nazwa_tabeli] WHERE [kolumna = 'warunek']</span>;</b>

Usunięcie konkretnego rekordu z tabeli - po kluczu głównym:
```sql
DELETE FROM `obiady` WHERE obiad_id = 3;
```
Usunięcie kilku rekordów:
```sql
DELETE FROM `obiady` WHERE cena > 13 AND cena < 22;
```
Usunięcie wszystkich rekordów:
```sql
TRUNCATE TABLE `obiady`;
```
***
## 9. Import/eksport bazy.
> Import (<b>Baza musi zostać wcześniej utworzona</b>):
```sh
mysql -u root -p -h localhost [nazwa_bazy] < [plik_do_importu].sql
```
> Eksport
```sh
mysqldump -u root -p [nazwa_bazy] > [plik_do_eksportu].sql
```
***
## 10. **SELECT** - Pobieranie danych.
### Podstawowe wybieranie:
```sql
SELECT * FROM czytelnik; -- Wszystkie pola
SELECT tytul, autor FROM ksiazka;
```
### Klauzula `WHERE`:
```sql
SELECT tytul, autor FROM ksiazka
WHERE rok_wydania = 2013;
```
### Operatory `AND`, `OR`, `NOT`:
```sql
SELECT tytul, autor, rok_wydania
FROM ksiazka
WHERE rok_wydania = 2013 AND stron > 435;

SELECT tytul, autor, rok_wydania
FROM ksiazka
WHERE rok_wydania = 2013 OR rok_wydania = 2015;

SELECT *
FROM zamowienie
WHERE data_odbioru IS NOT NULL;
```

### Inner join'y:
```sql
SELECT _c.imie, _c.nazwisko, _k.tytul
    FROM zamowienie as _z
INNER JOIN czytelnik as _c
    ON _z.id_czytelnik = _c.id_czytelnik
INNER JOIN ksiazka as _k
    ON _k.id_ksiazka = _z.id_ksiazka;
```

### Left join'y:
```sql
SELECT * 
FROM czytelnik as _c
LEFT JOIN zamowienie as _z
    ON _z.id_czytelnik = _c.id_czytelnik
WHERE _z.id_zamowienie IS NULL;
```
------------
# Przykładowe zadania:
1. Stwórz bazę o nazwie baza1.
2. Stwórz bazę danych o nazwie baza2 i przypisz jej domyślny zestaw znaków UTF8.
3. Wypisz wszystkie bazy danych.
4. Stwórz bazę danych o nazwie bazaRomana.
5. W bazie baza1 stwórz tabelę users i umieść w niej następujące pola:
    * id - <i>klucz główny</i>
    * imię i nazwisko - <i>nie może być puste</i>
    * email - <i>wartość unikalna</i>
    * wiek - <i>wymuszenie wartości > 17</i>
    * data_rejestracji
6. Dodaj do tabeli users 4 następujących użytkowników:
    * Adam Rogal, adamr232@gmail.com, 32 lata, 2014-10-13
    * Zbigniew Aleksandrowicz, za@zag.org, 59 lat, 2012-04-23
    * Piotr Quake, q@librotic.net, 72 lata, 1998-02-22
    * Hermenegilda  Konstantynopolitańczykiewiczówna, kk1998@gmail.com, 40, 2000-01-22
7. Wyświetl kolumny tabeli users.
8. Uaktualnij dane użytkownika Piotr Quake - zmień jego adres email na qauke_piotr@happy_pony.org.
9. Wybierz wszystkich użytkowników, których wiek wynosi 32 lata. Wykorzystaj WHERE.
10. Wybierz wszystkich użytkowników z wiekiem poniżej 50 lat.
11. Wybierz wszystkich użytkowników, którzy posiadają konto pocztowe na gmailu.
12. Usuń użytkownika Adam Rogal.
13. Ustaw atrybut wartości domyślnej dla email w tabeli users równy "nieznany"
14. Usuń tabelę users.
15. Stwórz własną tabelę produkty, wypełnij ją minimum 4 przykładowymi produktami. Ustal kolumny:
    * id,
    * nazwa
    * cena
    * kategoria produktu
    * vat
    * data dodania
    * sztuk