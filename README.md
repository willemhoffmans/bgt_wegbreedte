# Breedtescript BGT wegdelen

**Inleiding**

Doel van het breedtescript BGT wegdelen is de automatische berekening van de wegdelen zoals ze in de Basis Grootschalige Topografie (BGT) zijn opgenomen. De breedte van de wegdelen wordt vervolgens gekoppeld aan een lijnen dataset met wegdelen, zoals bijvoorbeeld het Nationaal Wegenbestand (NWB), OpenStreetMap of een eigen wegennetwerk. 

**Voorwaarden / uitgangspunten**

Om met het script te werken zijn een aantal dingen nodig:

  * Een PostgreSQL database, met PostGIS functionaliteit. Bij voorkeur PostgreSQL 10.x of hoger, PostGIS 2.4 of hoger.
  * Tabel met BGT wegdelen (vlakken), genaamd bgt.wegdeel
  * Tabel met een wegennetwerk, waaraan een unieke identifier gekoppeld is.

**Werken met het script**



