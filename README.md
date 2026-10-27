[README.md](https://github.com/user-attachments/files/34015756/README.md)
# Smart Appliance Energy Logger and Cost Calculator (ESP32 + Blynk)

Course project for **21AIE221T – Internet of Things: Architectures and Protocols**
SRM Institute of Science and Technology, Ramapuram, Chennai

**Authors:** Rakesh G (RA2411047020010), Mukilan B P (RA2411047020040)
**Guide:** Dr. Judy Flavia B, Assistant Professor, Department of AI & AIML

## Overview

Most people see their electricity use only as a monthly total. This project upgrades an existing ESP32 *Power Monitoring and Relay Control System* into an **appliance energy logger and cost calculator**. The **hardware is unchanged**; the upgrade is entirely in the firmware and the web dashboard.

The system:

- calculates power from the sensor readings and accumulates energy in kWh using the real time between samples: `kWh = P (W) × t (h) / 1000`
- calculates cost from an electricity rate that can be changed while the dashboard is running
- publishes energy and cost to Blynk virtual pins V3 and V4
- saves the energy total to flash so it survives a restart
- trips the relay automatically if power stays above a limit for 3 consecutive samples, or if cumulative energy reaches a limit
- shows everything on a browser dashboard that polls the Blynk REST API and falls back to a simulation mode after 3 failed polls

> This is a learning and demonstration prototype. It is **not** a billing meter or a protection device (it does not replace a fuse or circuit breaker).

## Architecture

Device layer (ESP32, PZEM-004T, relay) → Wi-Fi → Blynk Cloud (virtual pins) → browser dashboard (`index.html`, Chart.js).

The ESP32 samples every 1 s, integrates power, and pushes values. The dashboard polls every 2 s.

## Hardware

| Component | Specification |
|---|---|
| Microcontroller | ESP32 DevKit V1 |
| Voltage / current sensor | PZEM-004T |
| Relay | 1-channel, active-low module |
| Power supply | 5 V USB adapter |
| Network | 2.4 GHz Wi-Fi |

Relay GPIO in the report firmware is `RELAY_PIN = 26`; verify it against your physical build.

## Software requirements

- Arduino IDE 2.x
- Arduino core for ESP32 (includes the `Preferences` library)
- Blynk library (version compatible with your ESP32 core)
- A Blynk.Cloud account (template, datastreams, device token)
- Google Chrome (or any modern browser); Chart.js 4.4.1 is loaded from a CDN

## Blynk datastreams

| Pin | Name | Type | Direction |
|---|---|---|---|
| V1 | Voltage (V) | Double | Device → cloud (existing) |
| V2 | Relay | Integer 0/1 | Both ways (existing) |
| V3 | Energy (kWh) | Double, 3 decimals | Device → cloud (new) |
| V4 | Cost | Double, 4 decimals | Device → cloud (new) |
| V5 | Power (W) | Double | Device → cloud (optional) |
| V6 | Rate (per kWh) | Double | Cloud → device (optional) |
| V7 | Energy reset | Integer 0/1 | Cloud → device (optional) |

## Setup

1. **Wire the hardware** as in the base project (no changes). See the block diagram and circuit model in the report (Fig. 5.1, 5.2).
2. **Create the Blynk template** and add datastreams V1–V7 as in the table above. Note the device token.
3. **Flash the firmware.** Open the listing from *Appendix C* of the report in the Arduino IDE, set your Wi-Fi name, password and Blynk token, select *ESP32 Dev Module*, and upload.
4. **Open the dashboard.** Open `index.html` in a browser. Choose the data source:
   - *Simulation* (default, no token needed), or
   - *Live Blynk*: paste the device token and keep the server as `blynk.cloud`.
5. Set the electricity rate and press Apply. Use the relay switch to turn the appliance on and off.

## Firmware constants

| Constant | Default | Meaning |
|---|---|---|
| `RELAY_PIN` | 26 | GPIO for the relay |
| `RELAY_ACTIVE_LOW` | true | Low level switches the load on |
| `POWER_FACTOR` | 1.0 | Used in P = V × I × PF |
| `DEFAULT_RATE` | 0.15 | Cost per kWh until the dashboard sends a rate |
| `POWER_LIMIT_W` | 1500 | Power trip level |
| `OVERLOAD_SAMPLES` | 3 | Consecutive samples above the limit before a trip |
| `ENERGY_LIMIT_KWH` | 1.000 | Energy trip level |
| `SAMPLE_MS` | 1000 | Sampling interval |
| `SAVE_MS` | 300000 | Flash save interval (5 min) |

The hardware tests used `POWER_FACTOR = 0.95`, `POWER_LIMIT_W = 1000`, `ENERGY_LIMIT_KWH = 0.2` (0.01 for the energy-limit test) and a rate of ₹8 per kWh.

## Dashboard features

- Energy Consumption and Estimated Cost tiles, live power
- Chart of power and cumulative energy over time
- Editable rate and currency symbol
- Relay toggle, session reset, CSV download of logged samples
- Simulation mode with the same trip rule and speeds of 1x, 60x or 600x

## Results

Tested on a 10 W LED lamp, a 50 W table fan and a 65 W laptop charger, 30 minutes each, against a plug-in watt-meter.

| Metric | Value |
|---|---|
| Mean absolute energy error | 2.03 % |
| Largest absolute error | 2.08 % (table fan) |
| Cost check (dashboard cost − energy × rate) | 0 in all three tests |
| Trip delay, sustained overload | 3.1 s (design about 3 s) |
| Load change to dashboard | 2.85 s |
| Relay click to relay action | 0.65 s |
| Fallback to simulation after network loss | 6.00 s |

## Limitations

- One appliance, one sensor, one relay channel
- Constant power factor assumed, which adds error for some loads
- Small test loads and short runs, so percentage errors are sensitive to display rounding
- No multi-tariff pricing, taxes or fixed charges
- Dashboard chart needs Chart.js from a CDN (use a local copy for offline demos)

## Future work

Multiple appliances, time-of-use tariffs, a mobile app, and a per-sample latency study.

## Repository contents

- `firmware/` – ESP32 firmware (Appendix C of the report)
- `index.html` – web dashboard
- `docs/` – project report and presentation

## References

See the References chapter of the project report for the full list.
