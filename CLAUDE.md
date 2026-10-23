# CLAUDE.md

Project rules inherited from the parent folder `progetti`: code, comments, UI, README and commit messages in
English; answers to Francesco in Italian; commits and pushes only when he asks; never commit credentials
or real sample data; plots follow PlotStyleKit (`../PlotStyleKit`).

## What this is

Flash-photolysis setup (ESP32 + fast photodiode + ADC) to measure P+QA- charge recombination (about 100 ms)
of *Rb. sphaeroides* reaction centers at 865 nm with about 1 ms resolution. See `README.md` for the design,
the signal estimate and the bill of materials.

## State

Design phase only: nothing built, nothing measured. All numbers in the README are estimates from memory.
Do not present them as verified. Open questions are listed in the README (flash trigger level, flash duration,
absorbance at 865 nm, filter and LED parts).

## Decisions taken

- Detector: Si photodiode + transimpedance amplifier (or OPT101) + ADS1115/ADS1015, not an integrated
  light sensor like the TSL2591 (too slow, wrong spectral response).
- Cheap version first; no 865 nm interference filter, since signals are at least 50 mOD.
- The flash already exists and is triggered through BNC.

## Reuse

`../esp32-od-monitor` has the ESP32 firmware conventions, the flash-from-server tool and the MQTT/Influx/Grafana stack.
