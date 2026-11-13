# Verkeersmodel Amsterdam Viewer (VMA Viewer)

**Interactieve, projectgebonden visualisatie van verkeersmodelresultaten uit het Verkeersmodel Amsterdam (VMA).**

## Inhoud

- [Over het project](#over-het-project)
- [Functionaliteiten](#functionaliteiten)
- [Hoe werkt het?](#hoe-werkt-het)
- [Benodigdheden](#benodigdheden)
- [Configuratie](#configuratie)
- [Viewer genereren](#viewer-genereren)
- [Viewer gebruiken](#viewer-gebruiken)
- [Technische opbouw](#technische-opbouw)
- [Datadefinities en interpretatie](#datadefinities-en-interpretatie)
- [Beperkingen en aandachtspunten](#beperkingen-en-aandachtspunten)
- [Samenwerken en bijdragen](#samenwerken-en-bijdragen)
- [Licentie](#licentie)

## Over het project

De **VMA Viewer** is een hulpmiddel om resultaten van verkeersmodelberekeningen toegankelijk en inzichtelijk te maken. De viewer toont verkeersintensiteiten op een interactieve kaart, maakt het mogelijk verschillende modelvarianten te vergelijken en ondersteunt de analyse van kruispunten.

De huidige implementatie bestaat uit een **Ruby-exportscript**, bedoeld voor uitvoering in een omgeving met toegang tot de gegevens en scriptingfunctionaliteit van **OmniTRANS**. Het script leest gegevens uit het verkeersmodel, verwerkt deze en genereert één zelfstandig HTML-bestand waarin de benodigde modelgegevens en de viewer zijn opgenomen.

De gegenereerde HTML kan vervolgens worden geopend in een webbrowser. Voor de achtergrondkaart is een internetverbinding nodig; de geëxporteerde modelgegevens zijn in het HTML-bestand opgenomen.

### Toepassingen

De viewer is onder andere bruikbaar voor:

- het presenteren van modelresultaten in verkeersonderzoeken;
- het vergelijken van referentie-, autonome en planvarianten;
- het analyseren van verschillen in verkeersintensiteiten;
- het inzichtelijk maken van resultaten voor collega's en andere betrokkenen;
- het bekijken van kruispuntinformatie binnen een geselecteerd onderzoeksgebied.

## Functionaliteiten

### 1. Interactieve kaart

De resultaten worden weergegeven op een interactieve kaart op basis van **Leaflet**. De implementatie gebruikt het Nederlandse coördinatenstelsel **RD / EPSG:28992** en een grijze achtergrondkaart van **PDOK**.

De gebruiker kan inzoomen, uitzoomen en door het geselecteerde gebied navigeren. De export bevat geometrie van het relevante netwerk en de begrenzing van het onderzoeksgebied.

### 2. Verkeersintensiteiten per modaliteit

De viewer ondersteunt drie modaliteiten:

| Modaliteit | In de viewer |
| --- | --- |
| Gemotoriseerd verkeer | Wegvakintensiteiten |
| Openbaar vervoer | Intensiteiten van OV-reizigers |
| Fietsverkeer | Fietsintensiteiten, weergegeven in bandbreedtes |

De weergave en legenda worden aangepast aan de geselecteerde modaliteit.

### 3. Verschillende dagdelen

Voor iedere ondersteunde modaliteit kan worden gekozen uit:

| Dagdeel | Periode |
| --- | --- |
| Etmaal | 00:00–24:00 uur, gemiddelde werkdag |
| Ochtendspits | 07:00–09:00 uur |
| Avondspits | 16:00–18:00 uur |

De spitsperioden betreffen telkens twee uur. Houd bij de interpretatie rekening met het verschil tussen etmaal- en spitsintensiteiten.

### 4. Modelvarianten vergelijken

Er zijn twee weergavemodi:

**Eén variant bekijken** — toont de intensiteiten van één geselecteerde modelvariant.

**Twee varianten vergelijken** — toont verschillen tussen twee modelvarianten. De vergelijking wordt berekend als **vergelijkingsvariant minus basisvariant**. In de verschilweergave worden toenames rood en afnames groen weergegeven; basiswaarden worden afzonderlijk aangeduid.

Hiermee kunnen bijvoorbeeld de effecten van een ruimtelijk plan op het verkeersnetwerk inzichtelijk worden gemaakt.

### 5. Kruispuntanalyse

Bij de modaliteit **gemotoriseerd verkeer** kan een kruispuntlaag worden ingeschakeld. De exportcode bevat gegevensverwerking voor geregelde en ongeregelde kruispunten, inclusief kruispuntmetadata en intensiteiten.

De viewer bevat een detailweergave waarin kruispunttakken en informatie voor etmaal, ochtendspits en avondspits worden weergegeven. De precieze beschikbaarheid van gegevens hangt af van de gebruikte modelvarianten en selecties.

### 6. Projectinformatie

Elke export bevat metadata, waaronder:

- projectnummer en projectnaam;
- gebruikte VMA-versie;
- exportdatum;
- namen van de geëxporteerde modelvarianten.

Deze gegevens maken de HTML-export herkenbaar en herleidbaar naar de gebruikte modelinvoer.

## Hoe werkt het?

De verwerking bestaat uit twee delen:

```text
OmniTRANS / VMA-modelgegevens
              |
              v
       Ruby-exportscript
              |
              |  Selecties, netwerkgeometrie,
              |  intensiteiten en kruispuntgegevens
              v
        JSON-datastructuur
              |
              v
      Zelfstandig HTML-bestand
              |
              v
     Browser met VMA Viewer
              |
              v
  Kaart, varianten en analyses
```

Het Ruby-script voert via `OtQuery` SQL-query's uit op de modelgegevens. Daarna worden de resultaten samengebracht in een JSON-object (`trafficData`), dat rechtstreeks in de gegenereerde HTML wordt geplaatst. De browser voert de visualisatie en de vergelijking van varianten uit.

## Benodigdheden

### Voor het genereren

- Een werkende **OmniTRANS-omgeving** met het betreffende VMA-model.
- Toegang tot de benodigde modelvarianten en de relevante gegevens in de modeldatabase.
- Een Ruby-uitvoeromgeving waarin de OmniTRANS-objecten **`$Ot`** en **`OtQuery`** beschikbaar zijn.
- Een bestaande **area selection** voor het onderzoeksgebied.
- Optioneel een selectie van kruispunten.
- Schrijfrechten op de map waarin de export wordt opgeslagen.

Het script gebruikt de Ruby-standaardbibliotheken `date`, `json`, `csv` en `fileutils`. Een gewone losse Ruby-installatie is **niet voldoende** als de OmniTRANS-specifieke objecten en modelgegevens ontbreken.

### Voor het bekijken

- Een moderne webbrowser met JavaScript ingeschakeld.
- Internettoegang voor de PDOK-achtergrondkaart.

Voor het openen van een reeds gegenereerde viewer is geen OmniTRANS-installatie vereist.

## Configuratie

Bovenaan het Ruby-script staan de instelbare parameters. De onderstaande waarden zijn **voorbeelden uit de aangeleverde versie 0.8** en moeten worden aangepast aan het eigen project.

```ruby
vma_projectnummer = "250001"
vma_projectnaam = "Plan Woonwijk"
vma_versie = "5.1"

boundary_selection = "Selectie_Woonwijk"
kruispunt_selection = ""

varianten = [
  "VMA_2025AR_Referentie",
  "VMA_2035AR_Autonoom",
  "VMA_2035AR_Plan_A",
  "VMA_2035AR_Plan_B"
]
```

| Parameter | Betekenis |
| --- | --- |
| `vma_projectnummer` | Identificatie van het verkeersonderzoek of project |
| `vma_projectnaam` | Leesbare projectnaam in de export |
| `vma_versie` | Versie van het gebruikte Verkeersmodel Amsterdam |
| `boundary_selection` | Naam van de bestaande area selection die het exportgebied bepaalt |
| `kruispunt_selection` | Naam van de selectie met te exporteren kruispunten; kan leeg zijn |
| `varianten` | Array met exacte namen van de modelvarianten die moeten worden uitgelezen |

**Let op:** de selectie- en variantnamen moeten overeenkomen met de namen in de modelomgeving. De aangeleverde code geeft expliciet aan dat alleen de parameters bovenaan bedoeld zijn om te wijzigen.

## Viewer genereren

1. Open het Ruby-script in de daarvoor geschikte OmniTRANS-scriptingomgeving.
2. Controleer of de gewenste modelvarianten beschikbaar zijn.
3. Maak of controleer de area selection die het onderzoeksgebied afbakent.
4. Vul bovenaan het script het projectnummer, de projectnaam, de VMA-versie, de selecties en de modelvarianten in.
5. Voer het script uit in de omgeving waarin `OtQuery` beschikbaar is.
6. Controleer de uitvoermeldingen over het onderzoeksgebied, de gevonden netwerkobjecten en de opgehaalde intensiteiten.
7. Open het gegenereerde HTML-bestand in een browser en controleer de resultaten.

### Naam van het exportbestand

De bestandsnaam wordt automatisch opgebouwd volgens:

```text
<projectnummer>_<jjjj-mm-dd>_<projectnaam>.html
```

Spaties in de projectnaam worden vervangen door streepjes en de naam wordt omgezet naar kleine letters. Het bestand wordt geschreven naar de werkmap van de uitvoeromgeving. De exportdatum wordt automatisch ingevuld.

### Controle na export

Controleer in ieder geval of de kaart het juiste gebied toont, alle verwachte varianten beschikbaar zijn, de intensiteiten overeenkomen met bekende modeluitvoer en de kruispuntinformatie correct wordt weergegeven. Een technisch geslaagde export is niet automatisch een inhoudelijk gevalideerd resultaat.

## Viewer gebruiken

### Starten

Open het HTML-bestand in een browser. Het informatiescherm licht de werking en de beperkingen van de viewer toe. Kies **Start viewer** om de kaart te bekijken.

### Variant selecteren

Gebruik het zijmenu om te kiezen tussen één variant bekijken en twee varianten vergelijken. Selecteer daarna de gewenste modelvariant of de twee te vergelijken varianten.

### Modaliteit en dagdeel kiezen

Selecteer **Fietser**, **OV-reiziger** of **Gemotoriseerd verkeer** en kies vervolgens **Etmaal**, **Ochtendspits** of **Avondspits**. De kaart wordt aangepast aan de gekozen combinatie.

### Kaartlagen en kruispunten

Met de beschikbare selectievakjes kunnen intensiteiten en, bij gemotoriseerd verkeer, kruispunten worden weergegeven. Klik op een beschikbaar kruispuntobject om de detailinformatie te openen. Niet iedere kaartlaag is bij iedere modaliteit beschikbaar.

### Verschillen interpreteren

In de vergelijkingsmodus wordt het verschil tussen de twee gekozen varianten berekend. Een positieve waarde betekent een hogere intensiteit in de vergelijkingsvariant; een negatieve waarde betekent een lagere intensiteit.

## Technische opbouw

| Onderdeel | Techniek / implementatie |
| --- | --- |
| Exportlogica | Ruby |
| Modeltoegang | OmniTRANS-objecten, `OtQuery` en SQL |
| Dataoverdracht | JSON, opgenomen in HTML |
| Gebruikersinterface | HTML, CSS en JavaScript |
| Kaartbibliotheek | Leaflet |
| UI-styling | Bootstrap-stijlen |
| Coördinatenstelsel | RD / EPSG:28992 |
| Achtergrondkaart | PDOK BRT Achtergrondkaart, grijs |
| Eindproduct | Eén HTML-bestand per export |

De JSON-export bevat onder andere de volgende hoofdonderdelen:

| Sleutel | Inhoud |
| --- | --- |
| `meta` | Projectgegevens, exportdatum en variantnamen |
| `boundary` | Geometrie van de onderzoeksbegrenzing |
| `geo` | Netwerkgeometrie als GeoJSON FeatureCollection |
| `stats` | Wegvakintensiteiten |
| `vristats` | Gegevens van geregelde kruispunten |
| `krstats` | Gegevens van overige kruispunten |
| `krint` | Kruispuntintensiteiten |
| `haltes` | Geëxporteerde haltegegevens |
| `stappers` | Aanvullende OV-gerelateerde gegevens |

De implementatie bevat de bibliotheekcode en stijlen in het HTML-sjabloon. De kaartachtergrond wordt extern geladen via PDOK. De huidige versie is dus geen webapplicatie met een eigen server-API of gebruikersdatabase.

## Datadefinities en interpretatie

### Modelresultaten, geen tellingen

De getoonde intensiteiten zijn **berekende modelresultaten**, niet rechtstreeks gemeten verkeersintensiteiten. De kwaliteit en toepasbaarheid zijn afhankelijk van de onderliggende VMA-versie, invoergegevens, kalibratie en modelaannames.

### Afronding en presentatie

De toelichting in de viewer vermeldt dat gemotoriseerd verkeer op tientallen en OV-reizigers op honderdtallen worden afgerond. Fietsintensiteiten worden in bandbreedtes weergegeven. Daardoor zijn verschillen tussen weergegeven waarden niet altijd geschikt voor interpretatie op een zeer gedetailleerd niveau.

### Vergelijkbaarheid

Vergelijk bij voorkeur modelvarianten met consistente definities, netwerken, modelinstellingen en perioden. Controleer bij opvallende verschillen of een wijziging wordt veroorzaakt door het onderzochte plan, een netwerkverandering of verschillen in de modelinvoer.

## Beperkingen en aandachtspunten

**Versie 0.8 is een testversie.** De broncode waarschuwt dat de viewer foutieve informatie kan bevatten.

- **Afhankelijk van OmniTRANS:** het exportscript kan niet zonder de vereiste OmniTRANS-objecten en modeldatabase worden uitgevoerd.
- **Selecties vereist:** zonder geldige area selection kan geen export van het onderzoeksgebied worden gemaakt.
- **Statische export:** na het genereren worden de gegevens in het HTML-bestand niet automatisch bijgewerkt. Nieuwe modelresultaten vereisen een nieuwe export.
- **Achtergrondkaart online:** zonder internetverbinding kan de PDOK-achtergrondkaart ontbreken, ook al zijn de modelgegevens lokaal aanwezig.
- **Bestandsgrootte:** omdat modelgegevens en viewer samen in één HTML-bestand staan, kan een groot exportgebied een omvangrijk bestand opleveren.
- **Datadeling:** het HTML-bestand bevat daadwerkelijk geëxporteerde modelgegevens. Beoordeel daarom vóór publicatie of deze gegevens gedeeld mogen worden.
- **Kruispuntinformatie:** de volledigheid hangt af van beschikbare kruispuntdata en de opgegeven selecties.
- **Halteweergave:** de code bevat halte-export en een schakeloptie voor haltes, maar de actuele gebruikersinterface biedt geen zichtbare modaliteitskeuze die de betreffende `ox`-voorwaarde activeert. Beschouw dit niet zonder verdere test als werkende eindgebruikersfunctionaliteit.
- **Validatie:** controleer uitkomsten altijd tegen de oorspronkelijke modelresultaten voordat ze worden gebruikt in besluitvorming of publicaties.

## Samenwerken en bijdragen

Bijdragen aan de VMA Viewer zijn welkom. Gebruik bij voorkeur deze werkwijze:

1. Maak een **issue** aan voor een bug, verbeterpunt of nieuwe functie.
2. Maak een aparte **branch** voor de wijziging.
3. Houd wijzigingen zo veel mogelijk beperkt tot één onderwerp.
4. Test zowel de Ruby-export als de gegenereerde HTML-viewer met representatieve modeldata.
5. Dien een **pull request** in met een beschrijving van de wijziging en de uitgevoerde controles.

Vermeld bij een bugmelding zo mogelijk de scriptversie, gebruikte VMA-versie, geselecteerde modaliteit, dagdeel, vergelijkingsmodus, verwachte uitkomst en feitelijke uitkomst. Voeg geen vertrouwelijke modelgegevens of persoonsgegevens toe aan openbare issues.

### Mogelijke verdere ontwikkeling

Onderstaande punten zijn **suggesties voor toekomstige ontwikkeling**, geen claims over bestaande functies:

- geautomatiseerde validatie van exports;
- expliciete controles op ontbrekende varianten of selecties;
- verdere modularisering van Ruby, JavaScript en CSS;
- automatische tests voor scenariovergelijkingen;
- verbetering van de haltefunctionaliteit;
- documentatie van alle gebruikte databronnen en modelvelden;
- voorbeelddata waarmee de viewer zonder een echte VMA-database kan worden gedemonstreerd.

## Licentie

**Licentie: GNU Affero General Public License v3.0 (`AGPL-3.0-only`).**

De AGPL-3.0 is een copyleftlicentie. Wie de software verspreidt, moet onder de toepasselijke voorwaarden de bijbehorende broncode beschikbaar stellen. Bij een gewijzigde versie die via een netwerk aan gebruikers wordt aangeboden, geldt daarnaast een verplichting om die gebruikers toegang tot de bijbehorende broncode te bieden.

Dit betekent **niet** dat iedere privéwijziging automatisch openbaar op GitHub moet worden gepubliceerd. De exacte verplichtingen volgen uit de licentietekst.

Voeg het volledige officiële licentiebestand als **`LICENSE`** toe aan de repository. De vermelding in deze README vervangt dat bestand niet. Controleer vóór publicatie ook of je bevoegd bent om alle opgenomen code en modelgegevens onder deze licentie beschikbaar te stellen, en of de licenties van gebruikte componenten hiermee verenigbaar zijn.

Meer informatie: [GNU AGPL v3.0](https://www.gnu.org/licenses/agpl-3.0.html).