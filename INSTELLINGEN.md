# Instellingen van de testinstallatie

*Stand: 23 oktober 2026.* Volledig overzicht van alle Vaillant-instellingen van de [testinstallatie](README.md#mijn-testinstallatie): aroTHERM plus VWL 75/6, VWZ AI, sensoCOMFORT VRC 720f, uniSTOR plus 300 l, 50 l buffervat, vloerverwarming in één zone.

Gebruik dit als referentie, niet als voorschrift. Wat goed werkt hangt af van je huis, je afgiftesysteem en je hydrauliek.

## Waar stel je wat in?

| Niveau | Waar | Zichtbaar in Home Assistant? |
|---|---|---|
| **A. Via eBUS** | sensoCOMFORT (regelaar, ebusd `BASV3`) | Ja: `number.…` / `select.…` (lezen en schrijven), tijdprogramma's als `sensor.…` (alleen lezen) |
| **B. Via myVAILLANT** | App of cloud-integratie (myPyllant) | Ja, via de cloud. Deels dezelfde instellingen als A |
| **C. Installateursniveau** | Installateursmenu van de sensoCOMFORT / VWZ AI | **Nee**: niet uit te lezen via eBUS, alleen handmatig op de regelaar |
| **D. Hydrauliek** | Pompen en kranen in de installatie zelf | Nee |

Het is steeds dezelfde instelling in de regelaar: wijzig je hem via HA, de app of de sensoCOMFORT, dan geldt de laatste wijziging.

---

## A. Via eBUS (sensoCOMFORT)

In de tabel staat de ebusd-naam. In HA heet de entity `number.<prefix>_<naam>` of `select.<prefix>_<naam>`, in kleine letters (bij de testinstallatie is de prefix `heating_ebusd_basv3`).

### Verwarming (circuit 1, vloer)

| Instelling | Waarde | ebusd-naam | Toelichting |
|---|---|---|---|
| Stooklijn | **0,25** | `Hc1HeatCurve` | Was 0,4 bij de start; stap voor stap verlaagd. 0,2 bleek te laag (huis zakte langzaam af) |
| Zelflerende stooklijn | uit | `AdaptHeatCurve` | |
| Kamerinvloed | **active** | `Hc1RoomTempModulation` | Was expanded, zie [README](README.md#kamerinvloed-active-of-expanded) |
| Min. aanvoer verwarmen | 17 °C | `Hc1HeatingFlowTempMin` | |
| Max. aanvoer verwarmen | 45 °C | `Hc1HeatingFlowTempMax` | |
| Zomergrens (verwarmen uit boven) | 15 °C | `Hc1SummerTempLimit` | Gemiddelde buitentemperatuur |
| Extra aanvoertemperatuur | 0 K | `Hc1ExcessTemp` | De vloerbuffer-blueprint verhoogt dit tijdelijk |
| Verlaagd bedrijf | normal | `Hc1SetbackMode` | |
| Circuit aan | on | `Hc1Status` | |
| Buffer-laadoffset | 8 K | `BufferChargeOffset` | |
| Correctie buitenvoeler | 0 K | `OffsetOutsideTemp` | |

### Zone 1

| Instelling | Waarde | ebusd-naam | Toelichting |
|---|---|---|---|
| Modus | **manueel** | `Z1OpMode` | Constante temperatuur, het tijdprogramma wordt niet gebruikt |
| Gewenste temperatuur | **21 °C** | `Z1ManualTemp` | |
| Verlaagde temperatuur | 19 °C | `Z1SetbackTemp` | Niet in gebruik (modus manueel) |
| Vakantietemperatuur | 20 °C | `Z1HolidayTemp` | |
| Quick veto | 30 °C | `Z1QuickVetoTemp` | |
| Speciale functie | auto | `Z1SFMode` | |
| Kamervoeler | in de regelaar | `Z1RoomZoneMapping` = VRC700 | |
| Tijdprogramma (niet actief) | ma–vr 06:00–22:00, za 07:30–23:30, zo 07:30–22:00, 20 °C | `Z1Timer_*` | Alleen lezen |

### Warm water

| Instelling | Waarde | ebusd-naam | Toelichting |
|---|---|---|---|
| Basis-setpoint | **38 °C** | `HwcTempDesired` | De kit verhoogt tijdelijk: zonne-boost 54–58 °C, legionella 60 °C |
| Laadhysterese | 6 K | `CylinderChargeHyst` | Laadt pas onder 32 °C |
| Laadoffset | 10 K | `CylinderChargeOffset` | Aanvoer = boiler-setpoint + offset |
| Max. aanvoer warm water | 80 °C | `HwcFlowTempMax` | |
| Max. laadtijd | 90 min | `MaxCylinderChargeTime` | Geen `number`-entity; schrijven via MQTT |
| Blokkeertijd na laden | 30 min | `HwcLockTime` | Alleen lezen. Na een lading mag de boiler 30 min niet opnieuw laden |
| Modus | auto | `HwcOpMode` | Met tijdvenster |
| Tijdvenster laden | **13:00–17:00**, elke dag | `HwcTimer_*` | Alleen lezen. Valt samen met het boostvenster van de kit |
| Speciale functie | auto | `HwcSFMode` | "1x laden" = eenmalige lading, ook buiten het venster |
| Warm water aan | yes | `HwcEnabled` | |
| Parallel laden | uit | `HwcParallelLoading` | |
| Legionella van Vaillant | **uit** | `HwcLegionellaDay` | Legionella regelt de kit op zon. Tijdstip `HwcLegionellaTime` (12:50) is daardoor ongebruikt |
| Circulatiepomp | 06:00–22:00 | `CcTimer_*` | Alleen lezen |

### Koelen

| Instelling | Waarde | ebusd-naam |
|---|---|---|
| Koelen toegestaan op circuit 1 | on | `Hc1CoolingEnabled` |
| Koelmodus zone 1 | **uit** | `Z1CoolingOpMode` |
| Koel-setpoints (manueel / verlaagd / tijdprogramma) | 22,5 °C | `Z1CoolingManualTemp`, `Z1CoolingSetbackTemp`, `Z1CoolingTimer_*` |
| Min. aanvoer koelen | 18 °C | `Hc1CoolingFlowTempMin` |
| Dauwpuntbewaking / offset | aan / 0 K | `Hc1DewPointMonitoring`, `Hc1DewPointOffset` |

### Systeem

| Instelling | Waarde | ebusd-naam | Toelichting |
|---|---|---|---|
| Systeem uit | no | `GlobalSystemOff` | |
| Blokkering energiebedrijf geldt voor | heatpumpandbackup | `ESCO` | |
| Multifunctioneel relais | zone | `MultiRelaySetting` | |
| Stille modus: tijdvakken | **geen** | `SilentTimer_*` | Alleen lezen. Zie C: stille modus |
| Circuit 2 en 3 | uit | `Hc2*`, `Hc3*` | Niet aanwezig |

---

## B. Via myVAILLANT (cloud)

Deze instellingen zie je in de app en via de myVAILLANT-integratie. De meeste zijn dezelfde als in A.

| Instelling | Waarde | Gelijk aan (A) |
|---|---|---|
| Stooklijn | 0,25 | `Hc1HeatCurve` |
| Verwarming uit boven buitentemperatuur | 15 °C | `Hc1SummerTempLimit` |
| Min. aanvoertemperatuur | 17 °C | `Hc1HeatingFlowTempMin` |
| Warm water-modus | Tijdgestuurd | `HwcOpMode` = auto |
| Boost warm water (eenmalige lading) | uit, naar behoefte | `HwcSFMode` "1x laden" |
| Afwezigheid, handmatig koelen, ventilatieboost | uit | |

Gebruik de cloud alleen waar eBUS geen alternatief biedt. eBUS is lokaal en sneller.

---

## C. Installateursniveau (niet via eBUS uit te lezen)

Deze staan in het installateursmenu van de sensoCOMFORT (voor de VWZ AI / buitenunit). Ze zijn niet in Home Assistant te zien. Noteer zelf wat je instelt.

| Instelling | Waarde | Toelichting |
|---|---|---|
| Energie-integraal: startwaarde | **−100 °min** (minimum) | Hoe lager, hoe langer de WP wacht voordat hij start voor verwarmen. Minder starts, langere runs. De actuele waarde is wel uit te lezen: `sensor.ebusd_hmu_energyintegral` |
| Compressor-hysterese verwarmen | **10 K** | Was 5 K. Voorkomt een korte extra run na het boilerladen: de compressor loopt door van verwarmen naar warm water en terug |
| Boilerladen: modus | **balans** | |
| Stille modus (fluistermodus) | 40 % | Beperkt alleen het maximale toerental. **Let op:** bij de testinstallatie zijn geen tijdvakken ingesteld (zie A, systeem). Mogelijk is de stille modus daardoor feitelijk niet actief |
| Gebouwcircuitpomp (config. pomp verwarmen) | Auto | Ca. 1200 l/h |

---

## D. Hydrauliek (handmatig)

| Onderdeel | Stand | Toelichting |
|---|---|---|
| Bufferpomp | Auto | |
| Pomp bij mengunit/verdeler | stand 1 (laagste) | Was ongeveer stand 3. Lager gezet zodat de pomp minder warm water via het buffervat rondpompt: de vloer krijgt warmer water en het temperatuurverschil over de vloer steeg van ca. 1,2 naar 3,5 K |
| Zonethermostaten | badkamer helemaal open, één slaapkamer dicht, overige open | Laat een paar groepen altijd open, zodat de WP zijn warmte kwijt kan |

---

## Wijzigingen in de instellingen

| Datum | Wijziging | Waarom |
|---|---|---|
| 12 okt 2026 | Kamerinvloed expanded → active | Zon in de regelaarkamer zette de verwarming voor het hele huis uit |
| 20 okt 2026 | Compressor-hysterese 5 → 10 K | Geen korte run meer na het boilerladen |
| 20 okt 2026 | Verdelerpomp naar stand 1 | Minder kortsluiting via het buffervat |
| 20 okt 2026 | Stooklijn 0,25 → 0,2 | Test: kan het met lagere aanvoer? |
| 21 okt 2026 | Stooklijn 0,2 → 0,25 | Huis zakte 0,2–0,5 °C per dag af, badkamer het meest |
