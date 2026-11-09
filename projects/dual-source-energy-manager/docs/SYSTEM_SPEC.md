# Dual-Source Energy Manager | Phase 0 Specification

**Status:** Design only. No firmware or hardware control has been implemented.  
**Goal:** Define an ESP32-hosted software model of a PV array, battery, load and grid backup. Forecast weather/irradiance will be an external input; electrical measurements and energy flows will be simulated in the initial version.

## 1. System parameters

| Parameter | Phase 0 value |
|---|---:|
| PV array rating | 500 W |
| PV performance factor | 0.80 (combined temperature, wiring and conversion allowance) |
| Battery nominal capacity | 1,500 Wh (1.5 kWh) |
| Battery SoC window | 20–100% (300 Wh reserve; 1,200 Wh usable) |
| Initial SoC | 60% |
| Charge efficiency | 95% |
| Discharge efficiency | 95% |
| Battery charge power limit | 300 W |
| Battery discharge power limit | 300 W |
| Model time step | 15 minutes (96 steps per day) |

These are illustrative modelling assumptions, not measured equipment ratings or a final design.

## 2. Weather input concept

Configure the intended location and request hourly global tilted irradiance (GTI), ambient temperature and cloud cover from the Open-Meteo Forecast API. Begin with an assumed panel tilt of 25° and azimuth of 180° (north-facing in Open-Meteo's convention); keep both configurable.

For the initial South African model, use hourly weather values and hold each hourly value across four 15-minute model steps. Do not label the repeated values as 15-minute weather measurements. Fetch forecast data at boot and refresh periodically, rather than on every dashboard poll.

**API access caveat:** the free API tier is limited to non-commercial use. Open-Meteo lists promotional activities as an example of commercial use. Confirm permission before embedding a live feed in a public portfolio. Weather data attribution and the API-access terms are separate; follow the provider's attribution and licence requirements where applicable.

## 3. Illustrative 24-hour load profile

Each value is average load power in watts for the corresponding local clock hour, 00:00–23:00. Hold it for four model steps.

~~~cpp
const float hourly_load_w[24] = {
  45, 40, 40, 40, 45, 60,
  90, 100, 90, 80, 70, 70,
  70, 70, 70, 80, 100, 130,
  150, 140, 110, 80, 60, 50
};
~~~

**Daily energy:** 1,880 Wh (1.88 kWh).  
**Peak load:** 150 W.

This is an illustrative profile, not measured household demand.

## 4. Decision rules in plain English

1. Estimate available PV power from irradiance, the 500 W array rating and the 0.80 performance factor. Limit modelled PV power to the array rating.
2. Solar serves the load first.
3. If PV exceeds the load, use the excess to charge the battery, subject to the 300 W charge limit, 95% charge efficiency and 100% SoC ceiling. Any further excess is recorded as unused/curtailed PV in the initial model.
4. If PV is below the load, calculate the deficit. The battery supplies as much of that deficit as it can, subject to the 300 W discharge limit and 20% SoC floor.
5. Grid supply covers any remaining deficit.
6. Grid charging is disabled in Phase 0.
7. Clamp and validate SoC after every model step. Log PV energy, load energy, battery charge/discharge, grid import, curtailed PV and SoC.
8. The dashboard shows the source mix for each interval, not just a single source label.

## 5. Sensor and display interfaces

- **Weather adapter:** timestamp, irradiance, temperature and cloud-cover values.
- **PV model:** estimated PV power based on irradiance and fixed model assumptions.
- **Load model:** hourly load profile distributed over four 15-minute steps.
- **Battery model:** energy balance, power limits, efficiencies and SoC bounds.
- **Sensor adapters:** functions for PV power, battery voltage/current/power, load power and estimated SoC return model values initially. Future INA219 measurements may replace selected voltage/current reads.
- **Important:** an INA219 measures voltage and current; it does not directly measure battery SoC. SoC estimation requires current integration and a battery model.
- **Dashboard:** timestamp, weather input, PV power, load power, battery SoC and flow, grid contribution, curtailed PV, source status and 24-hour energy summary.
- **Optional:** ThingSpeak summary logging. Keep any write key out of source control.

## 6. RGB status concept

Where the selected ESP32 board has a controllable RGB LED:
- **Red:** grid import is required.
- **Blue:** battery is discharging and grid import is not required.
- **Green:** solar serves the load or charges the battery without grid import.
- **Off/white:** idle, startup or unavailable model data.

LED behaviour is a user-interface indication only; it is not a safety signal.

## 7. Phase 0 acceptance checks

- The 24 hourly load values total 1,880 Wh/day.
- The scheduler produces 96 model steps per day.
- Each hourly load/weather input maps to four 15-minute model steps.
- Battery SoC never falls below 20% or exceeds 100%.
- Charge/discharge efficiency and power limits are applied consistently.
- Grid import supplies any remaining demand after solar and battery contributions.
- All simulated electrical values are clearly labelled **SIMULATED** in the dashboard and portfolio.

## 8. Safety and portfolio disclosure

This is an educational simulation, not a certified energy controller. It must not control a real inverter, battery, grid connection or mains circuit. A future hardware implementation requires suitable isolation, protection, certified equipment, fail-safe design and qualified electrical review.

**Portfolio disclosure:** weather forecast input is planned; PV, load, battery and energy-flow values are simulated in the initial version. Sensor-ready architecture is planned; real sensor integration has not been implemented.
