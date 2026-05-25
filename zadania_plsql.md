zad1. Napisz blok PL/SQL, który wyświetli:

liczbę kursantów w siedzibie,
liczbę kursów w siedzibie,
liczbę wykładowców w siedzibie.
Wymagania:

użyj zmiennych,
użyj SELECT COUNT(*) INTO ...,
użyj DBMS_OUTPUT.PUT_LINE.
Przykład wyniku:

Liczba kursantów: 85
Liczba kursów: 8
Liczba wykładowców: 19

------------------------------------------------------------

DECLARE
    v_kursanci NUMBER;
    v_kursy NUMBER;
    v_wykladowcy NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_kursanci FROM kursanci;
    SELECT COUNT(*) INTO v_kursy FROM kursy;
    SELECT COUNT(*) INTO v_wykladowcy FROM wykladowcy;

    DBMS_OUTPUT.PUT_LINE('Liczba kursantów: ' || v_kursanci);
    DBMS_OUTPUT.PUT_LINE('Liczba kursów: ' || v_kursy);
    DBMS_OUTPUT.PUT_LINE('Liczba wykładowców: ' || v_wykladowcy);
END;
/


 zad2.  Łączna wartość umów dla Bydgoszczy

 
DECLARE
    v_laczna_wartosc NUMBER;
BEGIN
    SELECT NVL(SUM(r.cena), 0) INTO v_laczna_wartosc
    FROM umowy u
    JOIN kursy k ON u.kurs_id = k.kurs_id
    JOIN rodzaje r ON k.rodzaj_id = r.rodzaj_id
    WHERE u.miasto = 'BYDGOSZCZ';

    DBMS_OUTPUT.PUT_LINE('Łączna wartość umów dla BYDGOSZCZY: ' || v_laczna_wartosc || ' zł');
END;
/


 zad3.  Ocena liczby umów dla miasta
DECLARE
    v_miasto VARCHAR2(50) := 'BYDGOSZCZ';
    v_liczba_umow NUMBER;
BEGIN
    SELECT COUNT(*) INTO v_liczba_umow FROM umowy WHERE miasto = v_miasto;
 
    IF v_liczba_umow = 0 THEN
        DBMS_OUTPUT.PUT_LINE('Brak umów dla miasta');
    ELSIF v_liczba_umow < 50 THEN
        DBMS_OUTPUT.PUT_LINE('Mała liczba umów');
    ELSIF v_liczba_umow <= 100 THEN
        DBMS_OUTPUT.PUT_LINE('Średnia liczba umów');
    ELSE
        DBMS_OUTPUT.PUT_LINE('Duża liczba umów');
    END IF;
END;
/
 zad4. Lista kursów w siedzibie
BEGIN
    FOR r IN (
        SELECT k.kurs_id, r.nazwa AS nazwa_rodzaju, r.godz, r.cena, w.imie, w.nazwisko
        FROM kursy k
        JOIN rodzaje r ON k.rodzaj_id = r.rodzaj_id
        JOIN wykladowcy w ON k.wykladowca_id = w.wykladowca_id
    ) LOOP
        DBMS_OUTPUT.PUT_LINE('Kurs ' || r.kurs_id || ': ' || r.nazwa_rodzaju || ', ' || 
                             r.godz || 'h, ' || r.cena || ' zł, prowadzący: ' || 
                             r.imie || ' ' || r.nazwisko);
    END LOOP;
END;
/



zad5. Procedura raportująca umowy dla miasta
CREATE OR REPLACE PROCEDURE raport_umow_miasto(p_miasto IN VARCHAR2) IS
    v_liczba NUMBER;
    v_wartosc NUMBER;
    v_srednia NUMBER;
BEGIN
    SELECT COUNT(*), NVL(SUM(r.cena), 0), NVL(AVG(r.cena), 0)
    INTO v_liczba, v_wartosc, v_srednia
    FROM umowy u
    JOIN kursy k ON u.kurs_id = k.kurs_id
    JOIN rodzaje r ON k.rodzaj_id = r.rodzaj_id
    WHERE u.miasto = p_miasto;

    DBMS_OUTPUT.PUT_LINE('Raport dla miasta: ' || p_miasto);
    DBMS_OUTPUT.PUT_LINE('Liczba umów: ' || v_liczba);
    DBMS_OUTPUT.PUT_LINE('Łączna wartość umów: ' || v_wartosc || ' zł');
    DBMS_OUTPUT.PUT_LINE('Średnia wartość umowy: ' || ROUND(v_srednia, 2) || ' zł');
END;
/
-- Wywołanie: EXECUTE raport_umow_miasto('BYDGOSZCZ');

zad6. Funkcja zwracająca cenę kursu
BEGIN raport_umow_miasto('BYDGOSZCZ'); END;
 
CREATE OR REPLACE FUNCTION wartosc_kursu(p_kurs_id IN NUMBER) RETURN NUMBER IS
    v_cena NUMBER;
BEGIN
    SELECT r.cena INTO v_cena
    FROM kursy k
    JOIN rodzaje r ON k.rodzaj_id = r.rodzaj_id
    WHERE k.kurs_id = p_kurs_id;
 
    RETURN v_cena;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 0; 
END;
/
DECLARE v_cena NUMBER; BEGIN v_cena := wartosc_kursu(1); DBMS_OUTPUT.PUT_LINE('Cena kursu: ' || v_cena); END;
/
 
 zad 7. Obsługa wyjątków: wyszukiwanie kursanta
CREATE OR REPLACE PROCEDURE pokaz_kursanta(p_kursant_id IN NUMBER) IS
    v_imie kursanci.imie%TYPE;
    v_nazwisko kursanci.nazwisko%TYPE;
BEGIN
    SELECT imie, nazwisko INTO v_imie, v_nazwisko
    FROM kursanci
    WHERE kursant_id = p_kursant_id;
 
    DBMS_OUTPUT.PUT_LINE('Kursant: ' || v_imie || ' ' || v_nazwisko);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Nie znaleziono kursanta o ID: ' || p_kursant_id);
END;
/

zad 8.  Kursor jawny: szczegółowy raport umów
 
DECLARE
    CURSOR c_umowy IS
        SELECT u.umowa_id, k.imie, k.nazwisko, r.nazwa AS nazwa_kursu, r.cena
        FROM umowy u
        JOIN kursanci k ON u.kursant_id = k.kursant_id
        JOIN kursy ku ON u.kurs_id = ku.kurs_id
        JOIN rodzaje r ON ku.rodzaj_id = r.rodzaj_id
        WHERE u.miasto = 'BYDGOSZCZ';
    v_umowa c_umowy%ROWTYPE;
BEGIN
    OPEN c_umowy;
    LOOP
        FETCH c_umowy INTO v_umowa;
        EXIT WHEN c_umowy%NOTFOUND;
        DBMS_OUTPUT.PUT_LINE('Umowa ' || v_umowa.umowa_id || ' | ' || 
                             v_umowa.imie || ' ' || v_umowa.nazwisko || ' | ' || 
                             v_umowa.nazwa_kursu || ' | ' || v_umowa.cena || ' zł');
    END LOOP;



zad 9. Raport umów ze Szczecina
CREATE OR REPLACE PROCEDURE raport_umow_szczecin IS
BEGIN
    FOR r IN (
        SELECT u.umowa_id, k.imie, k.nazwisko, rod.nazwa AS nazwa_kursu, rod.cena, u.miasto
        FROM umowy u
        JOIN kursanciFilia k ON u.kursant_id = k.kursant_id
        JOIN kursyFilia kf ON u.kurs_id = kf.kurs_id
        JOIN rodzajeFilia rod ON kf.rodzaj_id = rod.rodzaj_id
        WHERE u.miasto = 'SZCZECIN'
    ) LOOP
        DBMS_OUTPUT.PUT_LINE('Umowa ' || r.umowa_id || ' | ' || 
                             r.imie || ' ' || r.nazwisko || ' | ' || 
                             r.nazwa_kursu || ' | ' || r.cena || ' zł | ' || r.miasto);
    END LOOP;
END;
/
-- Wywołanie: EXECUTE raport_umow_szczecin;

zad 10. Raport całej uczelni

 CREATE OR REPLACE PROCEDURE raport_uczelni IS
    -- Bydgoszcz
    v_b_liczba NUMBER := 0; v_b_suma NUMBER := 0;
    v_b_max_kurs VARCHAR2(100); v_b_pop_kurs VARCHAR2(100);

    -- Szczecin
    v_s_liczba NUMBER := 0; v_s_suma NUMBER := 0;
    v_s_max_kurs VARCHAR2(100); v_s_pop_kurs VARCHAR2(100);
BEGIN
    DBMS_OUTPUT.PUT_LINE('RAPORT UCZELNI');
    DBMS_OUTPUT.PUT_LINE('--------------------------------');

    -- ===== BYDGOSZCZ =====
    SELECT COUNT(*), NVL(SUM(r.cena), 0) INTO v_b_liczba, v_b_suma
    FROM umowy u JOIN kursy k ON u.kurs_id = k.kurs_id JOIN rodzaje r ON k.rodzaj_id = r.rodzaj_id
    WHERE u.miasto = 'BYDGOSZCZ';

    SELECT nazwa INTO v_b_max_kurs FROM (
        SELECT r.nazwa FROM rodzaje r ORDER BY r.cena DESC
    ) WHERE ROWNUM = 1;

    BEGIN
        SELECT nazwa INTO v_b_pop_kurs FROM (
            SELECT r.nazwa, COUNT(*) as ilosc FROM umowy u
            JOIN kursy k ON u.kurs_id = k.kurs_id JOIN rodzaje r ON k.rodzaj_id = r.rodzaj_id
            WHERE u.miasto = 'BYDGOSZCZ' GROUP BY r.nazwa ORDER BY ilosc DESC
        ) WHERE ROWNUM = 1;
    EXCEPTION WHEN NO_DATA_FOUND THEN v_b_pop_kurs := 'Brak danych'; END;

    DBMS_OUTPUT.PUT_LINE('Miasto: BYDGOSZCZ');
    DBMS_OUTPUT.PUT_LINE('Liczba umów: ' || v_b_liczba);
    DBMS_OUTPUT.PUT_LINE('Łączna wartość umów: ' || v_b_suma || ' zł');
    DBMS_OUTPUT.PUT_LINE('Najdroższy kurs: ' || v_b_max_kurs);
    DBMS_OUTPUT.PUT_LINE('Najpopularniejszy kurs: ' || v_b_pop_kurs);
    DBMS_OUTPUT.PUT_LINE('');

    -- ===== SZCZECIN (korzysta z synonimów do bazy filii) =====
    SELECT COUNT(*), NVL(SUM(rod.cena), 0) INTO v_s_liczba, v_s_suma
    FROM umowy u JOIN kursyFilia kf ON u.kurs_id = kf.kurs_id JOIN rodzajeFilia rod ON kf.rodzaj_id = rod.rodzaj_id
    WHERE u.miasto = 'SZCZECIN';

    SELECT nazwa INTO v_s_max_kurs FROM (
        SELECT nazwa FROM rodzajeFilia ORDER BY cena DESC
    ) WHERE ROWNUM = 1;

    BEGIN
        SELECT nazwa INTO v_s_pop_kurs FROM (
            SELECT rod.nazwa, COUNT(*) as ilosc FROM umowy u
            JOIN kursyFilia kf ON u.kurs_id = kf.kurs_id JOIN rodzajeFilia rod ON kf.rodzaj_id = rod.rodzaj_id
            WHERE u.miasto = 'SZCZECIN' GROUP BY rod.nazwa ORDER BY ilosc DESC
        ) WHERE ROWNUM = 1;
    EXCEPTION WHEN NO_DATA_FOUND THEN v_s_pop_kurs := 'Brak danych'; END;

    DBMS_OUTPUT.PUT_LINE('Miasto: SZCZECIN');
    DBMS_OUTPUT.PUT_LINE('Liczba umów: ' || v_s_liczba);
    DBMS_OUTPUT.PUT_LINE('Łączna wartość umów: ' || v_s_suma || ' zł');
    DBMS_OUTPUT.PUT_LINE('Najdroższy kurs: ' || v_s_max_kurs);
    DBMS_OUTPUT.PUT_LINE('Najpopularniejszy kurs: ' || v_s_pop_kurs);
    DBMS_OUTPUT.PUT_LINE('--------------------------------');

    -- ===== PODSUMOWANIE =====
    DBMS_OUTPUT.PUT_LINE('PODSUMOWANIE');
    DBMS_OUTPUT.PUT_LINE('Liczba wszystkich umów: ' || (v_b_liczba + v_s_liczba));
    DBMS_OUTPUT.PUT_LINE('Łączna wartość wszystkich umów: ' || (v_b_suma + v_s_suma) || ' zł');
END;
/
-- Wywołanie: EXECUTE raport_uczelni;
