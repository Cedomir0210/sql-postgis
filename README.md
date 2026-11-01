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
