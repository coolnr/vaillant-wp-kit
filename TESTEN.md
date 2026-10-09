# Proefdraaien: de kit testen zonder risico

Met proefdraaien laat je de blueprints een paar dagen meedraaien **zonder dat ze iets naar de warmtepomp sturen**. Ze nemen precies dezelfde beslissingen, maar schrijven naar TEST-helpers. Daarna vergelijk je wat de kit gedaan zou hebben met wat er echt gebeurde.

Je gebruikt hiervoor **dezelfde bestanden en dezelfde installatiestappen** als bij een gewone installatie. Je kiest alleen bij het aanmaken van de automations de TEST-helpers als doel. Zo test je ook meteen de installatie-instructies.

**Moet ik proefdraaien?**
- **Ja**, als je al eigen automations hebt die je warmtepomp sturen. Anders sturen twee automations tegelijk aan dezelfde boiler-setpoint.
- **Aangeraden**, als je opstelling afwijkt van de [testinstallatie](README.md#mijn-testinstallatie) of als je het voorzichtig wilt aanpakken.
- **Niet nodig**, als je nog geen eigen sturing hebt en je opstelling erop lijkt. Volg dan gewoon de [installatie in de README](README.md#installatie).

---

## Stap 0 – Back-up

Maak een back-up van Home Assistant: *Instellingen → Systeem → Back-ups*.

## Stap 1 – Bestanden kunnen bewerken

Je hebt toegang nodig tot de map `/config`, bijvoorbeeld via de app **Studio Code Server** of **File editor**.

## Stap 2 – Packages aanzetten

Open `configuration.yaml` en controleer of dit erin staat:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Staat er al een `homeassistant:`-blok? Zet de regel `packages:` er dan onder. Maak geen tweede `homeassistant:` aan. Maak daarna de map `/config/packages/` aan als die nog niet bestaat.

## Stap 3 – Package-bestanden plaatsen

Download van GitHub en zet in `/config/packages/`:
- [`vaillant_wp.yaml`](https://raw.githubusercontent.com/coolnr/vaillant-wp-kit/main/packages/vaillant_wp.yaml) (de basis)
- [`vaillant_wp_proefdraaien.yaml`](https://raw.githubusercontent.com/coolnr/vaillant-wp-kit/main/packages/vaillant_wp_proefdraaien.yaml) (de TEST-helpers)

Tip: klik met rechts op de link en kies *Link opslaan als…*, of open hem en kopieer de inhoud naar een nieuw bestand met dezelfde naam.

Open `vaillant_wp.yaml` en vervang:

| Zoek | Vervang door | Voorbeeld |
|---|---|---|
| `sensor.JOUW_NETVERMOGEN` | je netvermogen-sensor in W (afname +, teruglevering −) | `sensor.p1_meter_vermogen` |
| `heating_ebusd_basv3` | prefix van je regelaar | laten staan als die klopt |
| `ebusd_hmu` | prefix van je buitenunit | laten staan als die klopt |

## Stap 4 – Controleren en herstarten

Ga naar *Ontwikkelhulpmiddelen → YAML → Configuratie controleren*. Is die groen, kies dan **Herstarten**.

## Stap 5 – Controleer de nieuwe sensoren

Ga naar *Ontwikkelhulpmiddelen → Statussen* en zoek op `wp_`. Elk van deze moet een waarde hebben:

| Entity | Wat je ziet |
|---|---|
| `sensor.wp_netvermogen_excl_warmtepomp` | W, negatief bij zonne-overschot |
| `sensor.wp_elektrisch_vermogen` | W (niet kW) |
| `sensor.wp_boiler_temperatuur` | °C |
| `sensor.wp_boiler_laadtijd_vandaag` | uren |
| `sensor.wp_status` | Stand-by / Warm water / Verwarmen … |
| `sensor.wp_buitentemperatuur_gemiddeld`, `sensor.wp_kamertemperatuur_zone_1` | °C |

Had je al een eigen "netvermogen zonder warmtepomp"-sensor? Dan moet `sensor.wp_netvermogen_excl_warmtepomp` daar gelijk aan zijn.

## Stap 6 – Beginwaarden instellen

Zet via *Instellingen → Apparaten & diensten → Helpers*:

| Helper | Zet op |
|---|---|
| `input_datetime.vaillant_wp_laatste_legionella` | datum en tijd van de laatste keer dat de boiler 60 °C was. Laat je hem leeg, dan start de legionella-cyclus bij de eerste zon |
| `input_number.vaillant_wp_test_boiler_setpoint` | je huidige basis-setpoint, bijv. 38 |
| `input_select.vaillant_wp_test_sfmode` | `auto` |
| `input_number.vaillant_wp_test_vloer_boost` | 0 |

## Stap 7 – Blueprints importeren

Gebruik de **importknoppen** in de [README](README.md#installatie). Je HA vraagt per blueprint om bevestiging. Daarna staan ze onder *Instellingen → Automations & scènes → Blueprints*.

Lukt dat niet? Kopieer dan de map `blueprints/automation/vaillant_wp_kit/` handmatig naar `/config/blueprints/automation/` en kies op de Blueprints-pagina **⋮ → Herladen**.

## Stap 8 – Drie TEST-automations aanmaken

Klik per blueprint op **Automation aanmaken**. De meeste velden staan al goed via het package. Vul alleen in:

**Zonne-boost warm water**
- WP statuscode: `sensor.ebusd_hmu_rundatastatuscode`
- Boiler-setpoint: **`input_number.vaillant_wp_test_boiler_setpoint`** ← TEST

**Legionella op zon**
- WP statuscode: `sensor.ebusd_hmu_rundatastatuscode`
- Boiler-setpoint: **`input_number.vaillant_wp_test_boiler_setpoint`** ← TEST
- Speciale functie warm water: **`input_select.vaillant_wp_test_sfmode`** ← TEST

**Vloer als zonnebuffer**
- Zone: je zone-climate of `select.…_z1opmode`
- Extra aanvoertemperatuur: **`input_number.vaillant_wp_test_vloer_boost`** ← TEST

Zet de drempels, tijden en temperaturen gelijk aan wat je nu gebruikt. Geef de automations een naam die begint met **"TEST –"**, zodat je ze herkent.

> **Belangrijk:** in deze stap kies je **nergens** een `number.…`- of `select.…`-entity van ebusd. Dan kan de kit niets naar de warmtepomp sturen.

## Stap 9 – Laten draaien en vergelijken

Laat het minstens **3 tot 5 dagen met zon** draaien. Vergelijk dan op drie plekken:

1. **Grafiek.** Zet in een geschiedenisgrafiek naast elkaar:
   - je echte boiler-setpoint (`number.…_hwctempdesired`) en `input_number.vaillant_wp_test_boiler_setpoint`;
   - je echte `number.…_hc1excesstemp` en `input_number.vaillant_wp_test_vloer_boost`.
2. **Logboek.** Filter op "Vaillant WP-kit". Bij elke beslissing staat er een regel, bijvoorbeeld "Zonne-boost: setpoint naar 54 °C (nieuw)".
3. **Traces.** *Automations → TEST-automation → Traces*. Hier zie je per keer welke voorwaarde slaagde of faalde.

**Wat goed is:** de sprongen naar 54 °C, terug naar 38 °C, naar 60 °C (legionella) en de vloer-boost vallen op dezelfde momenten, met hooguit een paar minuten verschil.

**Verschillen die je kunt verwachten:**
- Gebruik je een tweede boost (bijv. 58 °C) of een weekend-boost? Die heeft geen TEST-tegenhanger, tenzij je er een tweede TEST-automation voor maakt.
- De TEST-legionella "ziet" de echte boiler. Haalt die 60 °C, dan rondt ook de TEST-cyclus af. Dat is zo bedoeld.

**Wijkt iets af?** Kijk in de trace welke voorwaarde anders uitviel. Meestal is het een drempel, tijd of temperatuur die niet gelijk staat aan je eigen automation.

## Stap 10 – Afronden

**Tevreden en overstappen:**
1. Zet je eigen warmtepomp-automations uit (schakelaar in de lijst).
2. Kies in de drie automations bij de doelen de echte ebusd-entities en haal "TEST –" uit de naam.
3. Volg een week de starts per dag en het zon-aandeel. Terug kan altijd: draai stap 1 en 2 om.
4. Verwijder `vaillant_wp_proefdraaien.yaml` uit `/config/packages/` en herstart.

**Stoppen zonder overstappen:** verwijder de drie TEST-automations en `vaillant_wp_proefdraaien.yaml`, en herstart. Het basis-package mag blijven staan, want het schrijft zelf nergens naar.
