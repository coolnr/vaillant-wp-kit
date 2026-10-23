# Vaillant WP-kit voor Home Assistant

**Laat je Vaillant-warmtepomp slimmer omgaan met je eigen zonnestroom, zonder hem onnodig vaak aan en uit te laten springen.**

*Versie 0.3 (12 oktober 2026). Afgeleid van een installatie die sinds mei 2026 dagelijks draait.*

---

## Wat doet het voor je?

Heb je zonnepanelen en een Vaillant-warmtepomp? Dan lever je overdag vaak stroom terug, terwijl je warmtepomp 's avonds of 's nachts stroom van het net haalt. Deze kit verschuift dat:

| Functie | In gewone woorden |
|---|---|
| **Zonne-boost warm water** | Is er stroom over, dan warmt de warmtepomp de boiler extra op (bijv. tot 54 °C). Die warmte heb je 's avonds nog. |
| **Legionella op zon** | De boiler moet ongeveer eens per week naar 60 °C tegen legionella. De kit doet dat op een zonnige middag in plaats van op een vast moment. |
| **Vloer als buffer** | Bij overschot wordt de vloerverwarming iets warmer gestookt. De vloer houdt die warmte vast, zodat de warmtepomp 's avonds minder hoeft. |
| **Thermostaatknoppen** (optioneel) | Een gewone thermostaatklok in Home Assistant voor de vloer en de boiler. |

**Waarom "zo min mogelijk aan en uit"?** Elke keer dat de compressor start, kost dat extra slijtage en rendement. Eenvoudige zonnesturingen zetten de warmtepomp vaak aan en uit zodra er een wolk voorbij komt. Deze kit houdt daar rekening mee: een run die loopt, maakt hij af.

**Wat blijft hetzelfde?** Je warmtepomp blijft zelf voor je comfort zorgen. De kit zet alleen tijdelijk een paar instellingen hoger en daarna weer terug. Valt Home Assistant uit, dan werkt je warmtepomp gewoon zoals altijd.

> **Over dit project**
> Dit is mijn eerste gedeelde project. De automations heb ik de afgelopen maanden voor mijn eigen warmtepomp gebouwd en stap voor stap verbeterd, met veel uitzoekwerk in grafieken en compressorstarts. Omdat meer mensen in mijn omgeving een Vaillant hebben, wil ik dat werk delen.
>
> Het draait bij mij dagelijks, maar het is getest op één installatie. Er kunnen dus gaten of fouten in zitten, zeker bij een andere regelaar, firmware of ebusd-configuratie. Twijfel je? Begin dan met proefdraaien (zie [Waar begin je?](#waar-begin-je)).
>
> Loop je ergens tegenaan, of heb je een verbetering? Open een [issue](https://github.com/coolnr/vaillant-wp-kit/issues). Elke melding helpt om de kit voor iedereen beter te maken.
>
> **Benieuwd hoe mijn warmtepomp is ingesteld?** In [INSTELLINGEN.md](INSTELLINGEN.md) staan alle instellingen van mijn installatie (stooklijn, warm water, compressor, pompen), met per instelling waar je hem verandert: via eBUS, de myVAILLANT-app of het installateursmenu.

---

## Inhoud

1. [Begrippen](#begrippen) – voor wie nieuw is
2. [Wat heb je nodig?](#wat-heb-je-nodig)
3. [Waar begin je?](#waar-begin-je) – direct installeren of eerst proefdraaien
4. [Stap A – eBUS-koppeling](#stap-a--ebus-koppeling-ebusd) (sla over als je al ebusd-entities in HA hebt)
5. [Installatie](#installatie)
6. [Werkt het?](#werkt-het)
7. [Problemen oplossen](#problemen-oplossen)
8. [Bijwerken en verwijderen](#bijwerken-en-verwijderen)
9. [Mijn testinstallatie](#mijn-testinstallatie)
   - **Alle instellingen van de warmtepomp: [INSTELLINGEN.md](INSTELLINGEN.md)**
10. [Voor gevorderden](#voor-gevorderden)

---

## Begrippen

| Woord | Betekenis |
|---|---|
| **eBUS** | Het tweedraads communicatienetwerk waarmee je Vaillant-onderdelen (buitenunit, regelaar, binnenmodule) met elkaar praten. |
| **eBUS-adapter** | Een klein kastje dat je op die eBUS aansluit, zodat je netwerk kan meelezen en -schrijven. |
| **ebusd** | Het programma dat de eBUS-berichten vertaalt naar leesbare waarden. Draait als app in Home Assistant. |
| **MQTT** | De "postbode" tussen ebusd en Home Assistant. In HA is dat de app **Mosquitto broker** plus de integratie **MQTT**. |
| **Entity** | Eén waarde of knop in Home Assistant, bijv. `sensor.…_hwcstoragetemp` (boilertemperatuur). |
| **Setpoint** | De gewenste temperatuur die je instelt. |
| **Package** | Een YAML-bestand dat in één keer een set sensoren en helpers toevoegt. |
| **Blueprint** | Een sjabloon voor een automation. Je vult een paar velden in en de automation is klaar. |
| **Laadhysterese** | De boiler gaat pas opwarmen als hij een aantal graden onder de setpoint zit (bij Vaillant standaard 6 graden). |
| **Netvermogen** | Wat je op dit moment van het net haalt (positief) of terug levert (negatief), in watt. |

---

## Wat heb je nodig?

- **Een Vaillant-warmtepomp met eBUS**, zoals een aroTHERM (plus), met een sensoCOMFORT-regelaar (VRC 720). Getest op de [opstelling hieronder](#mijn-testinstallatie).
- **Een eBUS-adapter**, bijv. van [ebusd.eu](https://adapter.ebusd.eu), aangesloten op de eBUS van je installatie. Twijfel je over het aansluiten? Vraag het je installateur.
- **Home Assistant** (OS of Supervised, voor de apps). Minimaal versie 2024.10; getest op 2026.9.
- **Een sensor voor je netvermogen in watt**: positief bij afname, negatief bij teruglevering. Bijvoorbeeld van je P1-meter of EVCC. Controle: als je zonnepanelen veel opbrengen, moet de waarde negatief zijn.
- **Toegang tot je configuratiebestanden**, bijv. met de app **File editor** of **Studio Code Server**.

---

## Waar begin je?

| Jouw situatie | Aanpak |
|---|---|
| Je hebt **nog geen** automations die je warmtepomp sturen, en je opstelling lijkt op de [testinstallatie](#mijn-testinstallatie) | **Direct installeren**: volg [Installatie](#installatie) en kies meteen de echte ebusd-entities. |
| Je hebt **al eigen automations** voor je warmtepomp | **Eerst proefdraaien** ([TESTEN.md](TESTEN.md)). Anders sturen twee automations tegelijk aan dezelfde knop. |
| Je opstelling **wijkt af** (andere regelaar, uniTOWER, radiatoren, meerdere zones) of je wilt het voorzichtig aanpakken | **Eerst proefdraaien** ([TESTEN.md](TESTEN.md)): een paar dagen kijken wat de kit zou doen, zonder dat hij iets naar de warmtepomp stuurt. |

Proefdraaien gebruikt dezelfde bestanden en stappen als een gewone installatie; je kiest alleen andere doelen. Overstappen naar echt is daarna een kwestie van de doelen omzetten.

---

## Stap A – eBUS-koppeling (ebusd)

*Zie je in Home Assistant al entities als `number.…_hwctempdesired` en `sensor.…_hwcstoragetemp`? Sla deze stap dan over.*

1. **MQTT:** installeer de app **Mosquitto broker** (*Instellingen → Apps → App-winkel*) en voeg daarna de integratie **MQTT** toe (*Instellingen → Apparaten & diensten*). Meestal stelt HA die zelf voor.
2. **ebusd-app:** ga naar *Instellingen → Apps → App-winkel → ⋮ → Repositories* en voeg toe: `https://github.com/LukasGrebe/ha-addons`. Installeer daarna de app **eBUSd**.
3. **ebusd instellen** (tabblad *Configuratie* van de app):

   | Instelling | Waarde | Waarom |
   |---|---|---|
   | Seed HA MQTT integration config | **aan** | Zorgt dat de ebusd-waarden vanzelf als entities in HA verschijnen |
   | `--device=` | `ens:<IP-van-je-adapter>:9999` | Voor een netwerkadapter. Een USB-adapter kies je in het veld erboven |
   | `--mqttjson` | | Waarden in het formaat dat de kit verwacht |
   | `--scanconfig` | | Herkent zelf welke Vaillant-onderdelen op de bus zitten |
   | `--configpath=` | `https://ebus.github.io/next/` | Nieuwste definities; nodig voor recente regelaars zoals de VRC 720f |
   | `--mqttvar=` | `filter-direction=r\|u\|^w,filter-name=` | **Belangrijk:** zonder deze regel krijg je alleen *meetwaarden* en geen *instelbare* entities (setpoints, modi). Dan kan de kit niets sturen |

4. **Start** de app en wacht 5 tot 10 minuten. Daarna verschijnen de ebusd-entities (*Instellingen → Apparaten & diensten → MQTT*).

---

## Installatie

### 1. Packages aanzetten

Open `configuration.yaml` (met File editor) en controleer of dit erin staat:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Staat er al een regel `homeassistant:`? Zet `packages:` er dan onder, ingesprongen. Maak geen tweede `homeassistant:` aan. Maak daarna de map `packages` aan in `/config`, als die er nog niet is.

### 2. Je eigen namen opzoeken

Ga naar *Ontwikkelhulpmiddelen → Statussen* en zoek op **`hwctempdesired`**. Je vindt iets als `number.ebusd_basv3_hwctempdesired`.
- Het stuk tussen `number.` en `_hwctempdesired` is je **regelaar-prefix**: vaak `ebusd_basv3`, bij de testinstallatie `heating_ebusd_basv3`.
- Zoek ook op **`powerconsumptionhmu`** voor de **buitenunit-prefix**: meestal `ebusd_hmu`.
- Zoek je **netvermogen-sensor** op, bijvoorbeeld van je P1-meter.

### 3. Basispackage plaatsen

1. Open [`packages/vaillant_wp.yaml`](https://raw.githubusercontent.com/coolnr/vaillant-wp-kit/main/packages/vaillant_wp.yaml), kopieer alle tekst, en maak in File editor een nieuw bestand `/config/packages/vaillant_wp.yaml` met die tekst.
2. Vervang in dat bestand (zoeken en vervangen):
   - `sensor.JOUW_NETVERMOGEN` → jouw netvermogen-sensor
   - `heating_ebusd_basv3` → jouw regelaar-prefix (als die anders is)
   - `ebusd_hmu` → jouw buitenunit-prefix (als die anders is)
3. Ga naar *Ontwikkelhulpmiddelen → YAML → Configuratie controleren*. Bij groen: **Herstarten**.
4. Zoek bij *Statussen* op `wp_`. Sensoren als `sensor.wp_boiler_temperatuur` en `sensor.wp_netvermogen_excl_warmtepomp` moeten een waarde hebben.

*Optioneel – thermostaatknoppen:* wil je een ronde thermostaatklok voor de vloer en de boiler? Plaats dan ook [`packages/vaillant_wp_thermostaat.yaml`](https://raw.githubusercontent.com/coolnr/vaillant-wp-kit/main/packages/vaillant_wp_thermostaat.yaml) op dezelfde manier. Pas daarin dezelfde prefix aan, plus `ebusd/basv3/` als je regelaar anders heet. **Niet doen als je al eigen thermostaat-entities voor je Vaillant hebt.**

### 4. Blueprints importeren

Klik per blueprint op de knop. Je eigen Home Assistant opent en vraagt of je hem wilt importeren.

| Blueprint | Importeren |
|---|---|
| Zonne-boost warm water | [![Importeer in Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcoolnr%2Fvaillant-wp-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fvaillant_wp_kit%2Fvaillant_zonne_boost_warm_water.yaml) |
| Legionella op zon | [![Importeer in Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcoolnr%2Fvaillant-wp-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fvaillant_wp_kit%2Fvaillant_legionella_op_zon.yaml) |
| Vloer als zonnebuffer | [![Importeer in Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcoolnr%2Fvaillant-wp-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fvaillant_wp_kit%2Fvaillant_vloer_zonbuffer.yaml) |

Werkt de knop niet? Kopieer dan de map `blueprints/automation/vaillant_wp_kit/` naar `/config/blueprints/automation/` en kies op de Blueprints-pagina **⋮ → Herladen**.

### 5. Automations aanmaken

Ga naar *Instellingen → Automations & scènes → Blueprints*, klik op een blueprint en kies **Automation aanmaken**. De meeste velden staan al goed via het package. Je vult alleen dit in (zoek op het woord in de rechterkolom):

| Blueprint | Veld | Kies (zoekwoord) |
|---|---|---|
| Zonne-boost | WP statuscode | `rundatastatuscode` |
| | Boiler-setpoint | `hwctempdesired` (de `number.`) |
| Legionella | WP statuscode | `rundatastatuscode` |
| | Boiler-setpoint | `hwctempdesired` (de `number.`) |
| | Speciale functie warm water ("1x laden") | `hwcsfmode` |
| Vloerbuffer | Zone | `z1opmode` of je zone-thermostaat |
| | Extra aanvoertemperatuur | `hc1excesstemp` |

Loop daarna de instellingen per blueprint na. Zorg dat ze passen bij je warmtepomp (zie stap 6). De lijsten met statuscodes ("boiler laden" en "verwarmen") staan standaard goed voor een aroTHERM plus. Sla op.

*Proefdraaien?* Kies bij de doelen (setpoint, speciale functie, extra aanvoertemperatuur) de TEST-helpers. Zie [TESTEN.md](TESTEN.md).

### 6. Warmtepomp en kit op elkaar afstemmen

- **Basis-setpoint warm water:** zet de warmtepomp op dezelfde waarde als de blueprints (standaard **38 °C**). Dat kan via `number.…_hwctempdesired` in HA of op de sensoCOMFORT.
- **Boostvenster = warmwater-tijdvenster** van je warmtepomp (bij de testinstallatie 13:00–17:00). Anders kan de warmtepomp niet laden als de kit boost.
- **Legionella:** gebruik je de legionella-blueprint, zet dan de legionella-functie van Vaillant zelf **uit** (`select.…_hwclegionelladay` → off). Anders doen ze het allebei.
- **Legionella-teller:** zet *Instellingen → Apparaten & diensten → Helpers →* "WP laatste legionella-cyclus" op de laatste keer dat je boiler 60 °C was. Weet je dat niet? Laat hem leeg; dan start de eerste cyclus bij de eerstvolgende zon.

---

## Werkt het?

- **Logboek:** elke beslissing van de kit staat erin, onder "Vaillant WP-kit". Bijvoorbeeld *"Zonne-boost: setpoint naar 54 °C (nieuw)"*.
- **Traces:** *Instellingen → Automations → klik een automation → Traces*. Hier zie je per keer waarom hij wel of niet iets deed.
- **Statuscode actueel?** `sensor.ebusd_hmu_rundatastatuscode` moet binnen een minuut omspringen als de warmtepomp start. Het package zet daarvoor zelf een hogere poll-prioriteit.
- **Compressorstarts:** `sensor.wp_compressorstarts_vandaag` laat zien hoe vaak de compressor vandaag startte. Eén tot drie keer per dag is goed.

Een dashboard zit nog niet in de kit; dat volgt.

---

## Problemen oplossen

| Probleem | Oorzaak en oplossing |
|---|---|
| Geen `number.…`- of `select.…`-entities van ebusd | De regel `--mqttvar=` ontbreekt in de ebusd-app (zie [Stap A](#stap-a--ebus-koppeling-ebusd)). |
| `wp_*`-sensoren staan op "niet beschikbaar" | De prefix of de netvermogen-sensor in `vaillant_wp.yaml` klopt niet. Controleer [stap 2 en 3](#2-je-eigen-namen-opzoeken). |
| Configuratie controleren geeft een fout bij `packages` | Waarschijnlijk staat er twee keer `homeassistant:` in `configuration.yaml`. Voeg ze samen. |
| Thermostaatklok staat op "onbekend" | Normaal na een herstart; de kit vraagt de waarden binnen een minuut opnieuw op. Blijft het zo, controleer dan of ebusd draait. |
| Statuscode loopt minuten achter | De automation "Vaillant WP – statuscode poll-prioriteit" uit het package draait niet, of de MQTT-topic klopt niet (`ebusd/hmu/RunDataStatuscode/get`). Werkt het nog niet goed genoeg, verlaag dan de poll-interval van ebusd (`--pollinterval`). |
| De kit boost nooit | Controleer: het netvermogen is negatief bij zon, het boostvenster valt samen met het warmwater-tijdvenster, en de boiler zit laag genoeg (onder boost − hysterese − 1 °C). De trace laat zien welke voorwaarde faalde. |
| Vreemd gedrag of extra starts | Draaien er nog andere automations die dezelfde instellingen sturen? Zet die uit, of begin met [proefdraaien](TESTEN.md). |

Kom je er niet uit? Open een [issue](https://github.com/coolnr/vaillant-wp-kit/issues) met je opstelling en een screenshot van de trace.

---

## Bijwerken en verwijderen

**Bijwerken:** blueprints werk je bij via *Blueprints → ⋮ → Blueprint opnieuw importeren*. Je automations houden hun instellingen. Packages werk je bij door de tekst opnieuw over te nemen; vergeet je eigen vervangingen niet. Zie [Wijzigingen](#wijzigingen) voor wat er nieuw is.

**Verwijderen:** verwijder de automations en blueprints, haal de bestanden uit `/config/packages/` en herstart. Je warmtepomp blijft gewoon werken. Zet alleen de boiler-setpoint terug op je normale waarde als hij op een boost-waarde is blijven staan.

---

## Mijn testinstallatie

De kit is ontwikkeld en getest op deze installatie. Lijkt de jouwe erop, dan is de kans groot dat alles direct werkt.

| Onderdeel | Type |
|---|---|
| Buitenunit | aroTHERM plus **VWL 75/6** 230 V (7,5 kW), lucht/water |
| Binnenmodule | **VWZ AI** besturingsmodule (eBUS-adres 76), driewegklep, losse cv-pomp |
| Regelaar | **sensoCOMFORT VRC 720f** met draadloze buitenvoeler (ebusd: `BASV3`, adres 15) |
| Internetmodule | myVAILLANT connect **VR 940f** |
| Warm water | **uniSTOR plus 300 liter** boiler (VIH RW 300) |
| Buffervat | 50 liter |
| Afgifte | vloerverwarming, één zone (Z1); koelen mogelijk |
| Verdeling | buffervat parallel (4 aansluitingen), verdeler met eigen pomp; per kamer een zonethermostaat die een stelmotor (NC) op de verdeler stuurt |
| Koppeling met HA | ebusd 26.1 via **MQTT**, instellingen zoals in [Stap A](#stap-a--ebus-koppeling-ebusd) |
| Home Assistant | 2026.9 |
| Zonnepanelen / netmeting | ca. 8 kWp, netvermogen via EVCC/P1-meter |

### Instellingen van de warmtepomp

De kit verandert in de regelaar alleen tijdelijk de boiler-setpoint, "1x laden" en de extra aanvoertemperatuur. De belangrijkste vaste instellingen:

| Wat | Instelling | ebusd-naam |
|---|---|---|
| **Zone 1 – modus** | manueel (constante temperatuur, geen tijdprogramma) | `Z1OpMode` |
| Zone 1 – temperatuur | 21 °C | `Z1ManualTemp` |
| Stooklijn | 0,25 (zelflerend uit) | `Hc1HeatCurve`, `AdaptHeatCurve` |
| Aanvoertemperatuur verwarmen | min 17 °C, max 45 °C | `Hc1HeatingFlowTempMin/Max` |
| Zomergrens (verwarmen uit boven) | 15 °C buiten (gemiddeld) | `Hc1SummerTempLimit` |
| Ruimtetemperatuur-invloed | **active** (was expanded, zie hieronder) | `Hc1RoomTempModulation` |
| **Warm water – modus** | auto, met tijdvenster **13:00–17:00** | `HwcOpMode` |
| Warm water – basis-setpoint | **38 °C** (de kit boost tijdelijk naar 54–60 °C) | `HwcTempDesired` |
| Laadhysterese | 6 K (laadt pas onder 32 °C bij setpoint 38) | `CylinderChargeHyst` |
| Legionella van Vaillant | **uit**: de kit regelt dit op zon | `HwcLegionellaDay` |
| Compressor-hysterese verwarmen | 10 K (installateursmenu, niet via eBUS) | – |
| Energie-integraal start | −100 °min (installateursmenu, niet via eBUS) | – |

**Alle instellingen**, ingedeeld naar waar je ze instelt (eBUS, myVAILLANT, installateursmenu, hydrauliek), staan in [INSTELLINGEN.md](INSTELLINGEN.md).

#### Kamerinvloed: active of expanded?

Deze instelling bepaalt hoe de kamertemperatuur bij de sensoCOMFORT de verwarming beïnvloedt. Bij de testinstallatie is hij van **expanded** naar **active** gezet.

| | Expanded | Active |
|---|---|---|
| Wat het doet | Stooklijn + kamercorrectie, **en** de verwarming gaat helemaal uit zodra de kamer met de regelaar ongeveer 0,2 °C te warm is | Stooklijn + kamercorrectie (circa 2,5 °C lagere aanvoer per graad te warm), maar **geen** volledige uitschakeling. Alleen de zomergrens schakelt nog uit |
| Goed als | De kamer met de regelaar representatief is voor het hele huis | De kamer met de regelaar zon krijgt of warmer is dan de rest |
| Nadeel | Zon in die ene kamer zet de verwarming voor het hele huis uit (bij de testinstallatie soms 20+ uur), terwijl kamers zonder zon afkoelen | Overdag wordt doorgestookt met lauw water. Dat geeft meer verbruik op zonuren, een gelijkmatiger huis, en in het tussenseizoen mogelijk iets meer starts |

Met *active* draait de warmtepomp 's middags vaker voor de vloer. De blueprints houden daar rekening mee: een zonnerun voor de boiler mag ook starten als de warmtepomp voor de verwarming draait. De Vaillant schakelt dan door naar warm water (voorrang), meestal zonder extra compressorstart. De vloerbuffer werkt met *active* ook beter, omdat de verwarming niet meer helemaal uitgaat.

**Zonethermostaten per kamer:** sluiten er veel groepen tegelijk, dan kan de warmtepomp zijn warmte minder kwijt en worden de runs korter. Laat bij voorkeur een paar groepen altijd open (bij de testinstallatie: badkamer).

**Waar stel je dit in?** Het zijn instellingen in de regelaar zelf. Je kunt ze wijzigen op de sensoCOMFORT (een deel alleen in het installateursmenu), een deel in de myVAILLANT-app, of via de ebusd-entities in Home Assistant. Het blijft dezelfde instelling; de laatste wijziging geldt.

**Andere opstelling?** Een uniTOWER, een andere regelaar (VRC 700/`BASV2`), meerdere zones of radiatoren kunnen andere entity-namen of ander gedrag geven. Proefdraaien laat snel zien of het werkt. Meld het in een issue, dan voegen we je opstelling toe aan de lijst.

---

## Voor gevorderden

### Bestanden

| Onderdeel | Bestand | Wat het doet |
|---|---|---|
| Basispackage | `packages/vaillant_wp.yaml` | Genormaliseerde `wp_*`-sensoren bovenop ebusd, helpers, en een automation die de statuscode poll-prioriteit 1 geeft (`?1` op de `/get`-topic) |
| Thermostaat (optioneel) | `packages/vaillant_wp_thermostaat.yaml` | 2 MQTT-climate-entities, 2 bridge-automations (verwarmen/koelen als 2 Vaillant-circuits), 1 automation die na een herstart de waarden opvraagt (`/get`-topics) |
| Proefdraaien (optioneel) | `packages/vaillant_wp_proefdraaien.yaml` | TEST-helpers (`input_number`/`input_select`) als veilig doel |
| Instellingen testinstallatie | `INSTELLINGEN.md` | Alle Vaillant-instellingen, per niveau (eBUS, myVAILLANT, installateur, hydrauliek) |
| Blueprints | `blueprints/automation/vaillant_wp_kit/` | Zonne-boost, legionella, vloerbuffer. De beslisregels staan in de beschrijving van elke blueprint |

### Entities uit het basispackage

| Entity | Inhoud |
|---|---|
| `sensor.wp_netvermogen_excl_warmtepomp` | Netvermogen min het WP-verbruik (W). De kern van de sturing |
| `sensor.wp_elektrisch_vermogen` | WP-verbruik in W (ebusd geeft kW) |
| `sensor.wp_boiler_temperatuur`, `sensor.wp_aanvoertemperatuur`, `sensor.wp_buitentemperatuur_gemiddeld`, `sensor.wp_kamertemperatuur_zone_1` | Genormaliseerde temperaturen |
| `sensor.wp_status` | Stand-by / Warm water / Verwarmen / Koelen / Ontdooien |
| `sensor.wp_boiler_laadtijd_vandaag` | `history_stats` op de statuscode (hwc-states) |
| `sensor.wp_compressorstarts_totaal`, `sensor.wp_compressorstarts_vandaag` | Totaal en dagteller (`utility_meter`) |
| `input_boolean.vaillant_wp_legionella_actief`, `input_datetime.vaillant_wp_laatste_legionella`, `input_boolean.vaillant_wp_vakantie` | Status van de legionella-cyclus en vakantiemodus |

### Ontwerpprincipes

- **Netvermogen zonder de WP zelf.** Anders ziet een draaiende compressor zijn eigen verbruik als "geen overschot", stopt de sturing de run, en start de compressor opnieuw zodra de zon terugkomt. Dat gaf in de praktijk 2 tot 3 starts per dag in plaats van 1.
- **Een run die loopt, wordt altijd afgemaakt.** Terugzetten gebeurt pas na de laadbeurt (WP-vermogen < 100 W, 2 min), met een check tegen valse "klaar" na een herstart.
- **Rekening houden met de laadhysterese.** Een nieuwe run alleen als de boiler onder boost − hysterese − 1 °C zit. Een lopende run wordt verlengd; dat kost geen extra start.
- **Statuscodes als inputs.** "Boiler laden" (`hwc_*`) en "verwarmen" (`heat_*`) zijn instelbare lijsten. Nieuwe run: WP stil **of** verwarmend. Terugzetten: niet tijdens een boilerlading (statuscode `hwc_*`, of vermogen > 300 W zonder `heat_*`-status, als buffer voor een achterlopende statuscode). Extra trigger zodra de statuscode `hwc_*` 1 minuut verlaat.
- **"Vandaag al geladen".** Na een laadbeurt alleen opnieuw boosten als de boiler flink gebruikt is.
- **Herstartbestendig.** Geen lange wachtstappen; vaste vangnetten (deadline, time-out, afbreektijd), zodat een setpoint nooit blijft hangen.
- **Comfort blijft bij de WP.** Alleen tijdelijke verhogingen, altijd terug naar de basis.
- **Doelen domein-onafhankelijk.** Acties kiezen `number.set_value` of `input_number.set_value` op basis van het gekozen doel. Daardoor werkt proefdraaien met dezelfde blueprints.

---

## Let op

- Deze kit **schrijft** via eBUS naar je regelaar (setpoints, speciale functie). Overleg bij twijfel met je installateur.
- Andere regelaars of firmware kunnen andere entities of opties hebben. Controleer eerst of alles uit het package een waarde geeft.
- Gebruik je de zonne-boost twee keer (bijv. 54 en 58 °C)? Zet dan bij beide automations dezelfde basis-setpoint.

## Licentie

MIT, zie [LICENSE](LICENSE). Gebruik op eigen risico. Niet gelieerd aan Vaillant.

## Wijzigingen

- **23 okt 2026 (documentatie):** nieuw [INSTELLINGEN.md](INSTELLINGEN.md), met verwijzing in de intro en de inhoudsopgave, met alle instellingen van de testinstallatie, ingedeeld naar eBUS, myVAILLANT, installateursmenu en hydrauliek. Instellingentabel in de README bijgewerkt (stooklijn 0,25, zomergrens 15 °C, zelflerend uit).
- **0.3 (12 okt 2026):**
  - Zonne-boost en legionella: een nieuwe run mag ook starten als de WP voor de verwarming draait. Terugzetten kijkt naar de boilerlading in plaats van alleen het WP-vermogen, met een extra trigger als de lading klaar is. Statuscodes zijn nu instelbare inputs.
  - Package: automation die de statuscode poll-prioriteit 1 geeft.
  - README: uitleg over kamerinvloed active of expanded, en over zonethermostaten.
  - Optioneel package met thermostaatknoppen (climate) voor zone 1 en de boiler, inclusief het opvragen van de waarden na een herstart.
  - README herschreven: uitleg voor beginners, ebusd-installatie, problemen oplossen, bijwerken en verwijderen.
  - Testinstallatie en instellingen van de warmtepomp gedocumenteerd.
- **0.2 (9 okt 2026):**
  - Proefdraaimodus: doelen mogen ook `input_number` of `input_select` zijn.
  - Elke beslissing komt in het logboek ("Vaillant WP-kit").
  - Vloerbuffer werkt ook met de ebusd-zonemodus (`select`), niet alleen met `climate`.
  - Legionella-blueprint zonder YAML-ankers, zodat hij beter leesbaar is.
  - Op GitHub, met importknoppen en `source_url`.
- **0.1 (8 okt 2026):** eerste versie.
