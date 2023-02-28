DROP FUNCTION IF EXISTS remove_all();

CREATE or replace FUNCTION remove_all() RETURNS void AS $$
DECLARE
    rec RECORD;
    cmd text;
BEGIN
    cmd := '';

    FOR rec IN SELECT
            'DROP SEQUENCE ' || quote_ident(n.nspname) || '.'
                || quote_ident(c.relname) || ' CASCADE;' AS name
        FROM
            pg_catalog.pg_class AS c
        LEFT JOIN
            pg_catalog.pg_namespace AS n
        ON
            n.oid = c.relnamespace
        WHERE
            relkind = 'S' AND
            n.nspname NOT IN ('pg_catalog', 'pg_toast') AND
            pg_catalog.pg_table_is_visible(c.oid)
    LOOP
        cmd := cmd || rec.name;
    END LOOP;

    FOR rec IN SELECT
            'DROP TABLE ' || quote_ident(n.nspname) || '.'
                || quote_ident(c.relname) || ' CASCADE;' AS name
        FROM
            pg_catalog.pg_class AS c
        LEFT JOIN
            pg_catalog.pg_namespace AS n
        ON
            n.oid = c.relnamespace WHERE relkind = 'r' AND
            n.nspname NOT IN ('pg_catalog', 'pg_toast') AND
            pg_catalog.pg_table_is_visible(c.oid)
    LOOP
        cmd := cmd || rec.name;
    END LOOP;

    EXECUTE cmd;
    RETURN;
END;
$$ LANGUAGE plpgsql;
select remove_all();

CREATE TABLE adresa (
    adresa_id SERIAL NOT NULL,
    stat_id INTEGER NOT NULL,
    cislo_budovy VARCHAR(100) NOT NULL,
    mesto VARCHAR(100) NOT NULL,
    ulice VARCHAR(100) NOT NULL,
    psc VARCHAR(100) NOT NULL
);
ALTER TABLE adresa ADD CONSTRAINT pk_adresa PRIMARY KEY (adresa_id);

CREATE TABLE osoba (
    osoba_id SERIAL NOT NULL,
    adresa_id INTEGER NOT NULL,
     osoba_typ_id INTEGER NOT NULL,
    cislo VARCHAR(100) NOT NULL,
    jmeno VARCHAR(100) NOT NULL,
    prijmeni VARCHAR(100) NOT NULL,
    email VARCHAR(100)
);
ALTER TABLE osoba ADD CONSTRAINT pk_osoba PRIMARY KEY (osoba_id);

CREATE TABLE osoba_typ (
    osoba_typ_id SERIAL NOT NULL,
    nazev VARCHAR(100) NOT NULL
);
ALTER TABLE osoba_typ ADD CONSTRAINT pk_osoba_typ PRIMARY KEY (osoba_typ_id);

CREATE TABLE planeta (
    planeta_id SERIAL NOT NULL,
    nazev VARCHAR(50) NOT NULL
);
ALTER TABLE planeta ADD CONSTRAINT pk_planeta PRIMARY KEY (planeta_id);

CREATE TABLE platba (
    platba_id SERIAL NOT NULL,
    platba_typ_id INTEGER NOT NULL,
    zasilka_id INTEGER NOT NULL,
    cas TIME NOT NULL,
    castka VARCHAR(100) NOT NULL,
    datum DATE NOT NULL
);
ALTER TABLE platba ADD CONSTRAINT pk_platba PRIMARY KEY (platba_id);

CREATE TABLE platba_typ (
    platba_typ_id SERIAL NOT NULL,
    nazev VARCHAR(50) NOT NULL
);
ALTER TABLE platba_typ ADD CONSTRAINT pk_platba_typ PRIMARY KEY (platba_typ_id);

CREATE TABLE pobocka (
    pobocka_id SERIAL NOT NULL,
    adresa_id INTEGER NOT NULL,
    cislo VARCHAR(100) NOT NULL,
    nazev VARCHAR(50) NOT NULL
);
ALTER TABLE pobocka ADD CONSTRAINT pk_pobocka PRIMARY KEY (pobocka_id);

CREATE TABLE raketa (
    raketa_id SERIAL NOT NULL,
    raketa_typ_id INTEGER NOT NULL,
    nazev VARCHAR(100) NOT NULL
);
ALTER TABLE raketa ADD CONSTRAINT pk_raketa PRIMARY KEY (raketa_id);

CREATE TABLE raketa_typ (
    raketa_typ_id SERIAL NOT NULL,
    nazev VARCHAR(50) NOT NULL
);
ALTER TABLE raketa_typ ADD CONSTRAINT pk_raketa_typ PRIMARY KEY (raketa_typ_id);

CREATE TABLE stat (
    stat_id SERIAL NOT NULL,
    planeta_id INTEGER NOT NULL,
    nazev VARCHAR(50) NOT NULL
);
ALTER TABLE stat ADD CONSTRAINT pk_stat PRIMARY KEY (stat_id);

CREATE TABLE zamestnanec (
    osoba_id INTEGER NOT NULL,
    pobocka_id INTEGER NOT NULL,
    osobni_cislo INTEGER NOT NULL,
    plat VARCHAR(100) NOT NULL,
    pracovni_pozice VARCHAR(50) NOT NULL,
    datum_nastupu DATE NOT NULL
);
ALTER TABLE zamestnanec ADD CONSTRAINT pk_zamestnanec PRIMARY KEY (osoba_id);
ALTER TABLE zamestnanec ADD CONSTRAINT uc_zamestnanec_osobni_cislo UNIQUE (osobni_cislo);

CREATE TABLE zasilka (
    zasilka_id SERIAL NOT NULL,
    zasilka_typ_id INTEGER NOT NULL,
    prijemce_id INTEGER NOT NULL,
    odesilatel_id INTEGER NOT NULL,
    zamestnanec_id INTEGER NOT NULL,
    pobocka_id INTEGER NOT NULL,
    raketa_id INTEGER NOT NULL,
    datum_doruceni DATE NOT NULL,
    datum_odesilani DATE NOT NULL,
    hmotnost VARCHAR(100) NOT NULL,
    popis VARCHAR(1000)
);
ALTER TABLE zasilka ADD CONSTRAINT pk_zasilka PRIMARY KEY (zasilka_id);

CREATE TABLE zasilka_typ (
    zasilka_typ_id SERIAL NOT NULL,
    nazev VARCHAR(50) NOT NULL
);
ALTER TABLE zasilka_typ ADD CONSTRAINT pk_zasilka_typ PRIMARY KEY (zasilka_typ_id);

CREATE TABLE raketa_pobocka (
    raketa_id INTEGER NOT NULL,
    pobocka_id INTEGER NOT NULL
);
ALTER TABLE raketa_pobocka ADD CONSTRAINT pk_raketa_pobocka PRIMARY KEY (raketa_id, pobocka_id);

ALTER TABLE adresa ADD CONSTRAINT fk_adresa_stat FOREIGN KEY (stat_id) REFERENCES stat (stat_id) ON DELETE CASCADE;

ALTER TABLE osoba ADD CONSTRAINT fk_osoba_adresa FOREIGN KEY (adresa_id) REFERENCES adresa (adresa_id) ON DELETE CASCADE;

ALTER TABLE platba ADD CONSTRAINT fk_platba_platba_typ FOREIGN KEY (platba_typ_id) REFERENCES platba_typ (platba_typ_id) ON DELETE CASCADE;
ALTER TABLE platba ADD CONSTRAINT fk_platba_zasilka FOREIGN KEY (zasilka_id) REFERENCES zasilka (zasilka_id) ON DELETE CASCADE;

ALTER TABLE pobocka ADD CONSTRAINT fk_pobocka_adresa FOREIGN KEY (adresa_id) REFERENCES adresa (adresa_id) ON DELETE CASCADE;
ALTER TABLE osoba ADD CONSTRAINT fk_osoba_osoba_typ FOREIGN KEY (osoba_typ_id) REFERENCES osoba_typ (osoba_typ_id) ON DELETE CASCADE;

ALTER TABLE raketa ADD CONSTRAINT fk_raketa_raketa_typ FOREIGN KEY (raketa_typ_id) REFERENCES raketa_typ (raketa_typ_id) ON DELETE CASCADE;

ALTER TABLE stat ADD CONSTRAINT fk_stat_planeta FOREIGN KEY (planeta_id) REFERENCES planeta (planeta_id) ON DELETE CASCADE;

ALTER TABLE zamestnanec ADD CONSTRAINT fk_zamestnanec_osoba FOREIGN KEY (osoba_id) REFERENCES osoba (osoba_id) ON DELETE CASCADE;
ALTER TABLE zamestnanec ADD CONSTRAINT fk_zamestnanec_pobocka FOREIGN KEY (pobocka_id) REFERENCES pobocka (pobocka_id) ON DELETE CASCADE;



ALTER TABLE zasilka ADD CONSTRAINT fk_zasilka_zasilka_typ FOREIGN KEY (zasilka_typ_id) REFERENCES zasilka_typ (zasilka_typ_id) ON DELETE CASCADE;
ALTER TABLE zasilka ADD CONSTRAINT fk_zasilka_raketa FOREIGN KEY (raketa_id) REFERENCES raketa (raketa_id) ON DELETE CASCADE;
ALTER TABLE zasilka ADD CONSTRAINT fk_zasilka_osoba FOREIGN KEY (prijemce_id) REFERENCES osoba (osoba_id) ON DELETE CASCADE;
ALTER TABLE zasilka ADD CONSTRAINT fk_zasilka_osoba_1 FOREIGN KEY (odesilatel_id) REFERENCES osoba (osoba_id) ON DELETE CASCADE;
ALTER TABLE zasilka ADD CONSTRAINT fk_zasilka_pobocka FOREIGN KEY (pobocka_id) REFERENCES pobocka (pobocka_id) ON DELETE CASCADE;
ALTER TABLE zasilka ADD CONSTRAINT fk_zasilka_zamestnanec FOREIGN KEY (zamestnanec_id) REFERENCES zamestnanec (osoba_id) ON DELETE CASCADE;



ALTER TABLE raketa_pobocka ADD CONSTRAINT fk_raketa_pobocka_raketa FOREIGN KEY (raketa_id) REFERENCES raketa (raketa_id) ON DELETE CASCADE;
ALTER TABLE raketa_pobocka ADD CONSTRAINT fk_raketa_pobocka_pobocka FOREIGN KEY (pobocka_id) REFERENCES pobocka (pobocka_id) ON DELETE CASCADE;

