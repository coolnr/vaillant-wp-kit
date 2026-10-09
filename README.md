# Vaillant WP-kit voor Home Assistant

Slimme zonnesturing voor een Vaillant-warmtepomp via eBUS. Doel: meer warm water en warmte op eigen zonnestroom, met **zo min mogelijk compressorstarts**.

*Versie 0.2 (oktober 2026), afgeleid van een productie-installatie die sinds mei 2026 draait.*

**Eerst veilig uitproberen?** Zie [TESTEN.md](TESTEN.md): laat de kit meedraaien zonder iets naar de warmtepomp te sturen.

> **Over dit project**
> Dit is mijn eerste gedeelde project. De automations heb ik de afgelopen maanden voor mijn eigen warmtepomp gebouwd en stap voor stap verbeterd, met veel uitzoekwerk in grafieken en compressorstarts. Omdat meer mensen in mijn omgeving een Vaillant hebben, wil ik dat werk delen.
>
> Het draait bij mij dagelijks, maar het is getest op één installatie. Er kunnen dus gaten of fouten in zitten, zeker bij een andere regelaar, firmware of ebusd-configuratie. Begin daarom met [proefdraaien](TESTEN.md).
>
> Loop je ergens tegenaan, of heb je een verbetering? Open een [issue](https://github.com/coolnr/vaillant-wp-kit/issues). Elke melding helpt om de kit voor iedereen beter te maken.

## Wat zit erin

| Onderdeel | Bestand | Wat het doet |
|---|---|---|
| Basispackage | `packages/vaillant_wp.yaml` | Vaste `wp_*`-sensoren bovenop ebusd, netvermogen zonder WP-verbruik, laadtijd vandaag, compressorstarts per dag, helpers voor legionella en vakantie |
| Proefdraaien (optioneel) | `packages/vaillant_wp_proefdraaien.yaml` | TEST-helpers, zodat de blueprints kunnen meedraaien zonder de warmtepomp aan te sturen |
| Blueprint | `vaillant_zonne_boost_warm_water.yaml` | Boiler-setpoint omhoog bij zonne-overschot, en veilig terug na de laadbeurt |
| Blueprint | `vaillant_legionella_op_zon.yaml` | Wekelijkse 60 °C-desinfectie, bij voorkeur op zon, met vangnet en time-out |
| Blueprint | `vaillant_vloer_zonbuffer.yaml` | Vloer als warmtebuffer: aanvoertemperatuur een paar graden omhoog bij overschot |

## Vereisten

- Vaillant aroTHERM (plus) met sensoCOMFORT / VRC 720 (**getest: BASV3**, zie [Mijn testinstallatie](#mijn-testinstallatie))
- eBUS-adapter + **ebusd-app** met MQTT-discovery
- Een sensor voor het **netvermogen in W** (afname positief, teruglevering negatief), bijv. van P1-meter of EVCC
- Home Assistant 2024.10 of nieuwer

## Installatie

1. **Package**
   - Download [`packages/vaillant_wp.yaml`](https://raw.githubusercontent.com/coolnr/vaillant-wp-kit/main/packages/vaillant_wp.yaml) en zet hem in `/config/packages/`.
   - Zet in `configuration.yaml`:
     ```yaml
     homeassistant:
       packages: !include_dir_named packages
     ```
   - Zoek en vervang in het bestand:
     - `sensor.JOUW_NETVERMOGEN` → je netvermogen-sensor
     - `heating_ebusd_basv3` → de prefix van je regelaar
     - `ebusd_hmu` → de prefix van je buitenunit
   - Herstart HA en controleer of de `wp_*`-sensoren een waarde hebben.
2. **Blueprints**: klik per blueprint op de knop hieronder. Je eigen Home Assistant opent en vraagt of je hem wilt importeren.

   | Blueprint | Importeren |
   |---|---|
   | Zonne-boost warm water | [![Importeer in Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcoolnr%2Fvaillant-wp-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fvaillant_wp_kit%2Fvaillant_zonne_boost_warm_water.yaml) |
   | Legionella op zon | [![Importeer in Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcoolnr%2Fvaillant-wp-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fvaillant_wp_kit%2Fvaillant_legionella_op_zon.yaml) |
   | Vloer als zonnebuffer | [![Importeer in Home Assistant](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fcoolnr%2Fvaillant-wp-kit%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fvaillant_wp_kit%2Fvaillant_vloer_zonbuffer.yaml) |

   Lukt de knop niet? Kopieer dan de map `blueprints/automation/vaillant_wp_kit/` handmatig naar `/config/blueprints/automation/`.
3. **Automations aanmaken** vanuit elke blueprint. De meeste entities staan al goed via het package. Kies zelf alleen nog:
   - statuscode: `sensor.ebusd_hmu_rundatastatuscode`
   - boiler-setpoint: `number.…_hwctempdesired`
   - speciale functie: `select.…_hwcsfmode`
   - extra aanvoertemperatuur: `number.…_hc1excesstemp`
   - zone: een `climate.…` of de ebusd-zonemodus `select.…_z1opmode`
4. **Stel de basis-setpoint in de WP zelf** gelijk aan die in de blueprints (standaard 38 °C).
5. **Start de legionella-teller**: zet `input_datetime.vaillant_wp_laatste_legionella` op de datum van de laatste keer dat de boiler 60 °C was.

## Mijn testinstallatie

De kit is ontwikkeld en getest op deze installatie. Lijkt die van jou erop, dan is de kans groot dat alles direct werkt.

| Onderdeel | Type |
|---|---|
| Buitenunit | aroTHERM plus **VWL 75/6** 230 V (7,5 kW), lucht/water |
| Binnenmodule | **VWZ AI** besturingsmodule (eBUS-adres 76) + muurbeugel, driewegklep, losse cv-pomp |
| Regelaar | **sensoCOMFORT VRC 720f** met draadloze buitenvoeler (ebusd: `BASV3`, adres 15) |
| Internetmodule | myVAILLANT connect **VR 940f** |
| Warm water | **uniSTOR plus 300 liter** boiler (VIH RW 300) |
| Buffervat | 50 liter |
| Afgifte | vloerverwarming, één zone (Z1); koelen mogelijk |
| Koppeling met HA | ebusd 26.1 via **MQTT** (eBUS-adapter → ebusd-app → MQTT) |
| Home Assistant | 2026.9 |
| Zonnepanelen / netmeting | ca. 8 kWp, netvermogen via EVCC/P1-meter |

**Andere opstelling?** Een uniTOWER, een andere regelaar (VRC 700/`BASV2`), meerdere zones of radiatoren kunnen andere entity-namen of gedrag geven. Proefdraaien laat snel zien of het werkt. Meld het in een issue, dan voegen we je opstelling toe aan de lijst.

## Ontwerpprincipes

- **Netvermogen zonder de WP zelf.** Anders ziet een draaiende compressor zijn eigen verbruik als "geen overschot", stopt de sturing de run, en start de compressor opnieuw zodra de zon terugkomt. Dat gaf in de praktijk 2-3 starts per dag in plaats van 1.
- **Een run die loopt, wordt altijd afgemaakt.** Terugzetten gebeurt pas na de laadbeurt.
- **Rekening houden met de laadhysterese.** De WP laadt pas zo'n 6 K onder de setpoint. Een boost heeft dus alleen zin als de boiler laag genoeg zit.
- **Herstartbestendig.** Geen lange wachtstappen en vangnetten op vaste tijden, zodat een setpoint nooit blijft hangen.
- **Comfort blijft bij de WP.** De automations verhogen alleen tijdelijk en zetten terug. Valt HA uit, dan werkt de warmtepomp gewoon zoals ingesteld.

## Let op

- Deze kit **schrijft** via eBUS naar je regelaar (setpoints, speciale functie). Overleg bij twijfel met je installateur.
- Andere regelaars of firmware kunnen andere entities of opties hebben. Controleer eerst of alles uit het package een waarde geeft.
- Gebruik je de zonne-boost twee keer (bijv. 54 en 58 °C)? Zet dan bij beide instanties dezelfde basis-setpoint.

## Licentie

MIT, zie [LICENSE](LICENSE). Gebruik op eigen risico. Niet gelieerd aan Vaillant.

## Wijzigingen

- **0.2 (9 okt 2026):**
  - Proefdraaimodus: doelen mogen ook `input_number` of `input_select` zijn.
  - Elke beslissing komt in het logboek ("Vaillant WP-kit").
  - Vloerbuffer werkt ook met de ebusd-zonemodus (`select`), niet alleen met `climate`.
  - Legionella-blueprint zonder YAML-ankers, zodat hij beter leesbaar is.
  - Op GitHub, met importknoppen en `source_url`, zodat blueprints later opnieuw te importeren zijn bij updates.
- **0.1 (8 okt 2026):** eerste versie.
