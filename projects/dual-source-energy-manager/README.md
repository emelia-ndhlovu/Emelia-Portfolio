# Dual-Source Energy Manager

**Portfolio project · Phase 0: system definition**

A software-first extension of my final-year hybrid solar/grid power project. The planned ESP32 application will model a PV array, battery storage and grid backup, then present the results through a dashboard hosted by the ESP32 itself.

## Current status

- **Current phase:** Phase 0 — system definition
- **Firmware:** Not written yet
- **Hardware:** Target ESP32 development board; exact board to be selected
- **Electrical data:** Simulated only
- **Dashboard:** Planned
- **Sensor integration:** Future phase

> **Simulation disclosure:** This is not a live solar installation or a certified energy controller. PV output, load demand, battery behaviour, state of charge and source selection are planned as modelled values. No real electrical switching is implemented.

## What the project is intended to demonstrate

- Obtain location-based solar-radiation and weather forecast inputs.
- Model a 500 W PV array and a 1.5 kWh battery.
- Prioritise solar, then battery storage, then grid supply.
- Track energy flows and battery state of charge over a 24-hour period.
- Serve a local dashboard from an ESP32.
- Keep sensor-reading functions separate so selected simulated values can later be replaced with hardware measurements.
- Optionally publish summary data to ThingSpeak.

## Phase 0 model baseline

| Parameter | Initial value |
|---|---:|
| PV array rating | 500 W |
| PV performance factor | 0.80 |
| Battery nominal capacity | 1,500 Wh |
| Battery SoC window | 20–100% |
| Initial battery SoC | 60% |
| Charge efficiency | 95% |
| Discharge efficiency | 95% |
| Battery charge/discharge power limit | 300 W |
| Model time step | 15 minutes |
| Steps per day | 96 |
| Illustrative daily load | 1,880 Wh |
| Illustrative peak load | 150 W |

These values are design assumptions for an interpretable first model, not measured system specifications.

See [the Phase 0 system specification](docs/SYSTEM_SPEC.md) for the load profile, equations, decision rules and acceptance checks.

## Planned architecture

~~~text
Open-Meteo forecast
        |
        v
Weather-data adapter ----> PV output model ----+
                                                |
Illustrative load profile --------------------> Energy manager
                                                |
Battery capacity / SoC model -----------------+
                                                |
                                  +-------------+-------------+
                                  |                           |
                            ESP32 web server            Optional logger
                                  |                       ThingSpeak
                                  v
                            Local dashboard
~~~

## Planned source priority

1. **Solar:** serve the load first; use excess PV to charge the battery.
2. **Battery:** cover a solar shortfall, subject to the charge/discharge limit and 20% reserve.
3. **Grid:** supply any remaining load that solar and the battery cannot meet.

Grid charging is disabled in the initial model. The dashboard should show the contribution of each source rather than forcing every interval into a single-source label.

## Weather data and attribution

Weather inputs are planned from the [Open-Meteo Forecast API](https://open-meteo.com/en/docs). The API does not require a key for its free tier, but that tier is limited to non-commercial use. Open-Meteo's terms list promotional activities as an example of commercial use. Confirm permission with the provider before embedding a live feed in a public portfolio.

Where permitted and where the data is displayed, provide the required attribution: **Weather data by Open-Meteo.com**. Weather data is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/); the API-access terms are separate. See [Open-Meteo terms](https://open-meteo.com/en/terms).

## Sensor-ready design

Sensor adapter functions will initially return model values. A future hardware iteration may use an INA219 for voltage and current measurements. The INA219 does not measure SoC directly; SoC estimation will require a battery model and current integration, with appropriate calibration and validation.

## Safety and limitations

This is an educational software model. It must not control a real inverter, battery, grid connection or mains circuit. A future hardware version would require suitable isolation, overcurrent protection, certified equipment, fail-safe design and qualified electrical review.

## Roadmap

- [x] Define initial parameters, 24-hour load profile and plain-English decision rules.
- [ ] Implement and test the model independently of hardware.
- [ ] Add validated weather-data input.
- [ ] Host a responsive dashboard on the ESP32.
- [ ] Add optional ThingSpeak summary logging and board-specific RGB status.
- [ ] Replace selected simulated readings with real sensor measurements in a safely validated hardware setup.

## Project navigation

- [Portfolio project page](index.html)
- [Phase 0 system specification](docs/SYSTEM_SPEC.md)
- [Main portfolio](../../projects.html)
