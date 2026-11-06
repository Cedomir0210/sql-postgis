# sql-postgis
PostgreSQL/PostGIS
# Inženjerski SQL i PostGIS - Dnevnik učenja

## Dan 1: Postavljanje prostorne baze i inženjersko filtriranje podataka

**Cilj:** Migracija vektorskih GIS podataka (Shapefile) u PostgreSQL relacijsku bazu podataka i izvođenje prostornih analiza (računanje površine, filtriranje i agregacija) direktno kroz SQL kod, bez ručnog rada u QGIS-u.

### 1. Inicijalizacija baze i uspostavljanje konekcije
- Kreirana je lokalna baza `gis_projekti` (PostgreSQL 16, port 2209).
- Baza je nadograđena u prostorni sustav izvršavanjem komande:
  ```sql
  CREATE EXTENSION postgis;

## Dan 2: Napredna prostorna analiza - 2D Hidraulički model reke Rače

**Cilj:** Migracija HEC-RAS rezultata (poplavni talas) u PostgreSQL i izračunavanje tačne površine poplavljene šume za svaku parcelu, koristeći napredne prostorne preseke i agregaciju podataka.

### 1. Vektorizacija i uvoz hidrauličkih podataka
HEC-RAS raster sa dubinama vode je uspešno vektorizovan i uvezen u `public` šemu baze, zajedno sa vektorskim slojem šumskih parcela za sliv reke Rače (`terrain_poli_reka_raca`).

### 2. Upravljanje GIS strukturom i "Case-Sensitive" sintaksom
Ustanovljeno je da PostgreSQL strogo razlikuje velika i mala slova. Kolona za dubinu poplavnog talasa je prilikom uvoza iz QGIS-a definisana velikim slovima (`DN`), kao i samo ime tabele. Ovo je rešeno striktnom upotrebom dvostrukih navodnika u kodu (`p."DN"` i `"ime_tabele"`), čime je obezbeđen stabilan rad baze.

### 3. Fizički presek geometrija (ST_Intersection) i agregacija (GROUP BY)
Umesto sporog manuelnog sečenja u klasičnim GIS alatima, napisan je automatizovan SQL upit koji "u letu" rešava najteže zadatke. 
Upit prvo izoluje samo one parcele koje se fizički dodiruju sa vodom (`ST_Intersects`), zatim seče geometriju na mestu preklapanja (`ST_Intersection`), transformiše ih u inženjerski EPSG:32634 i računa potopljene hektare.

Primena funkcije `SUM` uz klauzulu `GROUP BY` je poslužila za spajanje (agregaciju) hiljada sitnih isečenih komadića, tako da svaka parcela u finalnom izveštaju ima tačno jedan red sa ukupnom površinom pod vodom.

```sql
SELECT
    s.fid AS parcela_id,
    SUM(ST_Area(ST_Transform(ST_Intersection(s.geom, p.geom), 32634)) / 10000) AS ukupno_potopljeno_ha
FROM
    terrain_poli_reka_raca AS s
JOIN
    "poli_poplava_divcibare_ndvi_change_detection" AS p
ON
    ST_Intersects(s.geom, p.geom)
WHERE
    p."DN" > 0
GROUP BY


Optimizacija performansi: Kreirani su GiST (Generalized Search Tree) prostorni indeksi nad geometrijama šumskih parcela i poplavnog talasa, čime je vreme izvršavanja kompleksnog ST_Intersects upita redukovano sa ~6 minuta na 82 milisekunde.







    s.fid;
