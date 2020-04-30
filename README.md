# Breedtescript BGT wegdelen

## Inleiding

Doel van dit script is de automatische berekening van de breedte van de wegdelen zoals ze in de Basis Grootschalige Topografie (BGT) zijn opgenomen. De breedte van de wegdelen kan vervolgens worden gekoppeld aan een lijnen dataset met wegvakken, zoals bijvoorbeeld het Nationaal Wegenbestand (NWB), OpenStreetMap of een eigen wegennetwerk. 

**WAARSCHUWING!**

Bruikbaarheid van de resultaten van dit script hangt af van inhoudelijke overwegingen die de gebruiker zich stelt bij het formuleren van de vraag "hoe breed is een weg?". Dit kan voor elke specifieke toepassing weer anders zijn. De gebruiker dient dan ook zeer kritisch naar de resultaten te kijken, en zo nodig het script aan te passen. 

## Korte uitleg
Het script voert de volgende acties uit:

 1. Een wegvak in de wegen dataset (lijnen) wordt in 10 delen van gelijke lengte opgeknipt.
 2. Op elke knip wordt een 'meetstreepje' dwars op de lengterichting gecreëerd, hetgeen resulteert in 9 meetstreepjes per wegvak.
 3. De breedte van de weg op deze plek wordt bepaald door de lengte van de doorsnijding van het meetstreepje met het BGT wegdeel. Ook wordt hieraan het type BGT wegvak (bgt functie) meegegeven.
 4. Vervolgens worden per wegvak over alle meetstreepjes de statistieken berekend: gemiddelde, minimale, maximale breedte, en de standaarddeviatie. Dit gebeurt altijd per type BGT wegvak, zodat bijvoorbeeld een lokaal wegdeel en aanliggend fietspad niet als één wegdeel wordt gerekend. 
 5. Vervolgens wordt een 'operationele breedte' berekend, waarbij uitbijters (afwijking > 1 STD) buiten beschouwing worden gelaten. De uitbijters zelf worden gemarkeerd met 'L' (breedte wijkt meer dan 1 STD naar beneden af) of 'H' (breedte wijkt meer dan 1 STD naar boven af). 

## Uitgangspunten

Om met het script te werken zijn een aantal dingen nodig:

  * Een PostgreSQL database, met PostGIS functionaliteit. Bij voorkeur PostgreSQL 10.x of hoger, PostGIS 2.4 of hoger.
  * Tabel met actuele, bestaande BGT wegdelen (vlakken), genaamd bgt.wegdeel. Deze heeft mimimaal de kolommen "geometrie_vlak" en "bgt_functie".
  * Tabel met een wegennetwerk, waaraan een unieke identifier gekoppeld is. Deze tabel heeft ook een geometrie kolom "geom".

## Werken met het script
Bedienen van het script gaat in twee stappen:

1. Draaien van het SQL script in PostgreSQL (pgAdmin, DB Manager in QGIS, of psql). Hierbij wordt in de database de functie "wh_wegbreedte_bgt_generiek" aangemaakt. 
2. Daarna kan de functie worden aangeroepen, waarbij een aantal argumenten moeten worden opgegeven.

SELECT wh_wegbreedte_bgt_generiek('wegennetwerk', 'wegvak_kolom', false, 0)


**Argumenten:**

* wegennetwerk: tabel met het wegennetwerk (lijnen) waarvoor de breedtestatistieken moeten worden berekend
* wegvak_kolom: kolom in het wegennetwerk met de unieke identifiers (wegvakken)

De laatste twee argumenten zijn voor testdoeleinden en worden bij operationeel gebruik niet benut. 

Voorbeeld:

SELECT wh_wegbreedte_bgt_generiek('nwb_lunetten', 'wvk_id', false, 0);

## Resultaten
Het script maakt resultaten aan in het schema "breedte_analyse_nieuw". Zo nodig wordt dat schema eerst aangemaakt. Hierin komen een aantal tabellen:

* wegennetwerk: een kopie van het oorspronkelijke wegennetwerk
* gemiddeldebreedte: wegennetwerk voorzien van breedte statistieken
* dwarslijntjes_uniek: tabel met 'meetstreepjes' waarmee de breedte wordt berekend. Voorzien van kolommen met lengte en afwijking (aanduiding of een meetstreepje buiten de STD valt)


## Testmodus
Wanneer het script onverwacht gedrag vertoont (foutmeldingen of ondeugdelijke resultaten) kan met behulp van de testmodus het gedrag worden geanalyseerd. Je kan het script dan een berekening laten uitvoeren voor één wegvak, en er blijven ook een aantal extra tussenresultaten bewaard. 

Voorbeeld aanroepen in testmodus:

SELECT wh_wegbreedte_bgt_generiek('nwb_lunetten', 'wegvak_kolom', true, 2345667);

## Voorbeeld resultaat

Hier een voorbeeld van een resultaat, ingezoomd op Utrecht Lunetten:

![shetlands](https://github.com/willemhoffmans/bgt_wegbreedte/blob/master/img/shetlands.png "Shetlands Utrecht")

Het wegvak is in roze weergegeven, de dwarslijntjes (met lengte) in lichtgroen. Er is één 'uitbijter' van 17,6 meter, en deze heeft veel invloed op de gemiddelde lengte (G = 6.56 meter, standaarddeviatie D = 4.16 meter). Na uitsluiting van dwarslijntjes die meer dan 1 STD afwijken krijg je de "operationele breedte": 5.18 meter. Het afwijkende lijntje wordt rood gemarkeerd (afwijking = H). 

