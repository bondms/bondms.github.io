---
title: Electric Vehicle Charging
---

# Electric Vehicle Charging

These notes are for the UK.

## Charging networks:

These are all non-subscription prices. Subscription services may allow cheaper per-unit pricing.

* ChargePoint: [https://www.chargepoint.com/](https://www.chargepoint.com/)
    * Usage: App or RFID.
    * Registration: Required.
    * Pricing: Variable. Some free for an initial period.
    * Notes:
        * £10 charged and added to account on first use of non-free charger.
        * First RFID is provided free of charge on request.
        * Phone's NFC can be used.
* Ecotricity/Electric Highway: [https://www.ecotricity.co.uk/](https://www.ecotricity.co.uk/)
    * Usage: App.
    * Registration: Not required; guest access available with the app.
    * Pricing: 30 p/kWh.
    * Notes:
        * Maximum 45 minutes charge per session.
* engenie: [https://engenie.co.uk/](https://engenie.co.uk/)
    * Usage: Contactless.
    * Registration: Not required.
    * Pricing: 36 p/kWh.
* Franklin Energy RAPID LiFe EV: [https://franklinenergy.co.uk/rapid-life-ev/](https://franklinenergy.co.uk/rapid-life-ev/)
    * Usage: App or RFID (Plugsurfing / Octopus Juice).
    * Registration: Required.
* GeniePoint: [https://www.geniepoint.co.uk/](https://www.geniepoint.co.uk/)
    * Usage: App, web or RFID.
    * Registration: Required for RFID. Guest access available with app and web.
* Instavolt: [https://instavolt.co.uk/](https://instavolt.co.uk/)
    * Usage: Contactless.
    * Registration: Not required.
    * Pricing: 35 p/kWh.
* Ionity: [https://ionity.eu/](https://ionity.eu/)
    * Usage: Web.
    * Registration: Not required.
    * Pricing: 69 p/kWh
* Pod Point: [https://pod-point.com/](https://pod-point.com/)
    * Usage: App or web.
    * Registration: Required.
    * Pricing: Variable. Some free.
    * Notes:
        * 15 minutes free during which time use app/web to confirm continuation.
* Polar Instant: [https://www.polarinstant.com/](https://www.polarinstant.com/)
    * Usage: App.
    * Pricing: £1.20 connection fee + potential per-unit charge.
    * Notes:
        * Requires minimum £10 initial credit.
* Shell: [https://www.shell.co.uk/motorist/ev-charging.html](https://www.shell.co.uk/motorist/ev-charging.html)
    * Usage: Contactless, app, RFID.
    * Registration: Required only for RFID.
    * Pricing: 39 p/kWh.

## Jargon

### Charging modes

* Mode 1: Use of a standard domestic socket without safety protection. Limited to 8 Amp / 2 kW. Not suitable for most EVs.
* Mode 2: Use of a stardard domestic socket or a commando socket with an in-cord control and protection device. Limited to 10 Amp / 2.4 kW for domestic sockets, 12 Amp from a commando socket.
* Mode 3: AC fast charging up to 22 kW. May be 1 or 3 phase. A typical 1 phase home charger is limited to 7 kW.
* Mode 4: DC fast or rapid charging.

### Connectors

* Type 2: Current European standard supporting 1 and 3 phase AC from 3 to 50 kW.
* CCS-combo: Combines a type-2 connector with additional pins for fast DC charging.
* CHAdeMO: An alternative connector originally popular with Japanese manufacturers
* Type G (AKA "granny cable"): 3-pin domestic socket.

## Cable limitations

All DC chargers and some AC chargers are tethered. In this case, the tethered cable will be appropriate for the charger, and the charging speed will be the best supported by the combination of the vehicle and the charger.

Some AC chargers, however, are untethered and require the user to provide a cable. In this case, cable selection may limit charging speed below the maximum supported by the combination of the vehicle and the charger.

Consider a user with the following two cables (both being type 2, mode 3):

* 3-phase, 16 Amp (11 kW).
* 1-phase, 32 Amp (7.2 kW).

Although the first cable is rated to a higher power, there are cases where using the second cable may provide a fast charging rate (such as in the first of these scenarios):

* 7 kW, 1-phase charger:
    * The first cable provides only 3 kW. The 16 Amp limitation prevents full use of the one available phase.
    * The second cable provides the full 7 kW.
* 11 kW, 3-phase charger:
    * The first cable provides the full 11 kW.
    * The second cable provides 3.6 kW.
* 22 kW, 3-phase charger:
    * The first cable provides 11 kW. The cable's 16 Amp limit applies to each phase.
    * The second cable provides 7.2 kW. The cable's single phase prevents use of the the other two phases.

To avoid these potential charging-speed limitations, a more versatile cable could be used that provides 3-phasea at 32 Amps. This would avoid the need to carry or select between multiple cables, but such a cable is likely to be bigger, heavier, and more expensive than either of the other options.

## Home charging

### Tariffs

Octopus offer a range of good tariffs with some that are especially good for EV owners. Octopus Go and Octopus Agile both offer cheap off-peak electricity, so are ideal if you can schedule your charging to fit in with off-peak times.

Octopus referral code: [share.octopus.energy/topaz-sheep-824](https://share.octopus.energy/topaz-sheep-824)

### Charging speeds

The rate of charging at rapid DC public charging station is dependent on many factors and therefore often unpredictable. Temperature, state of charge, whether other users are using nearby charger, etc. can all affect public charging rates.

Home charging, however, is much more predictable. Here are some examples:

#### Assumptions:

* 78 kWh usable battery capacity and 100% charging efficiency. In practice, a 78 kWh battery will actually have less usable capacity and will charge at less than 100 % efficiency, but the differences should approximately cancel out for these rough estimates.
* Driving economy: 280 kWh per 1000 miles. Adjust the calculations for your own economy which will be affected by car, driving style, weather, traffic, type of roads, etc.
* 4 hours off off-peak electricity per night at 5 pence per kWh, as per [Octopus Go](https://octopus.energy/go/).

#### 6 Amps (typically the lowest configurable charging rate):

* Power: `6 Amps * 240 Volts = 1.44 kW`
* Range gain rate: `1.44 kW / 280 kWh per 1000 miles * 1000 miles = ~5 mph`
* Nightly energy gain: `1.44 kW * 4 h = 5.76 kWh`
* Nightly percentage gain: `5.76 kWh / 78 kWh * 100 =  ~7 %`
* Nightly range gain: `5.76 kWh / 280 kWh per 1000 miles * 1000 miles = ~20 miles`

#### At 10 Amps (maximum rate available from a 13 Amp domestic socket):

* Power: `10 Amps * 240 Volts = 2.4 kW`
* Range gain rate: `2.4 kW / 280 kWh per 1000 miles * 1000 miles = ~8.5 mph`
* Nightly energy gain: `2.4 kW * 4 h = 9.6 kWh`
* Nightly percentage gain: `9.6 kWh / 78 kWh * 100 =  ~12 %`
* Nightly range gain: `9.6 kWh / 280 kWh per 1000 miles * 1000 miles = ~35 miles`

#### At 32 Amps (typical 1-phase home EV charger):

* Power: `32 Amps * 240 Volts = 7.68 kW` (Actual rating is typically 7.4 kW).
* Range gain rate: `7.4 kW / 280 kWh per 1000 miles * 1000 miles = ~26.5 mph`
* Nightly energy gain: `7.4 kW * 4 h = 29.6 kWh`
* Nightly percentage gain: `29.6 kWh / 78 kWh * 100 =  ~38 %`
* Nightly range gain: `29.6 kWh / 280 kWh per 1000 miles * 1000 miles = ~100 miles`
