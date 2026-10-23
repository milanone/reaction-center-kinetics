# Reaction-center kinetics

A small flash-photolysis setup to measure the charge-recombination kinetics of photosynthetic
reaction centers (*Rhodobacter sphaeroides*) at 865 nm, built around an ESP32.

**Status: design phase. Nothing is built or tested yet.** Everything below is a plan and
back-of-the-envelope estimates (made from memory of datasheets, not measured). Verify part
specifications before ordering.

## Goal

After a saturating actinic flash, the special pair P865 is oxidised and bleaches near 865 nm.
The P+ QA- state then recombines with a relaxation time of about **100 ms**. We want to record
that absorbance transient with about **1 ms** time resolution.

Typical sample: reaction centers in a standard cuvette, signals of **at least 50 mOD**
(not low-concentration work).

## Why not the TSL2591 used in `esp32-od-monitor`

- Minimum integration time is 100 ms, and it integrates over windows: far too slow.
- Other integrated light sensors do not fit either: TCS34725 has an IR-cut filter, OPT4001/OPT4048
  have a photopic (visible) response, AS7341 has only a broad NIR channel. None is selective at 865 nm.

So the detector is a **fast silicon photodiode + transimpedance amplifier + fast ADC**.

## Planned design

1. **Measuring beam**: IR LED at 860-870 nm (FWHM about 30-40 nm), run at low, constant
   current (10-30 mA) so that it does not drive the reaction centers. Constant-current source
   (e.g. LM317 with one resistor, or op-amp + transistor) fed by a low-noise linear regulator
   or a bench supply. Warm up 1-2 min before each series; LED output drifts about 0.1-0.3 %/degC.
2. **Actinic flash**: already available, triggerable through a BNC input. The ESP32 drives the
   trigger; the interface (transistor or optocoupler) must match the flash trigger level.
   The ESP32 works at 3.3 V and the flash probably wants 5 V (**to be confirmed**).
3. **Detector**: Si photodiode (BPW34 or similar) with a transimpedance amplifier, or an OPT101
   (photodiode and amplifier in one chip). A long-pass filter (830-850 nm) in front blocks
   the flash and fluorescence. Detector at 90 degrees to the flash. The gain should put the
   baseline near mid-scale of the ADC.
4. **ADC**: ADS1115 (16 bit, 860 SPS, about 1.2 ms per point, enough for a 100 ms decay)
   or ADS1015 (12 bit, 3300 SPS) for a finer step. The internal ESP32 ADC is too noisy and nonlinear.
5. **Acquisition**: on trigger, record a pre-trigger baseline plus 200-500 samples (0.2-0.5 s),
   normalise to the pre-trigger mean (dI/I), ignore the first millisecond (flash artefact,
   the detector saturates for tens of microseconds), then send the trace in one burst.
6. **Averaging** over 4-16 flashes, with a pause of a few seconds between flashes so that the
   centers return to the ground state. Then fit an exponential decay.

## Signal estimate

50 mOD is a transmission drop of about **11 %** (10^0.05 = 1.12). With a 12-bit ADC and the
baseline at mid-scale this is about 200-250 counts against a noise of a few counts, so the
signal-to-noise ratio per point is about 50-100 on a single flash. This is a theoretical
estimate and has not been measured.

## Rough bill of materials (estimates)

| Item | Cost |
|---|---|
| Si photodiode (BPW34 or similar) | 1-2 EUR |
| Transimpedance amplifier (low-noise op-amp + passives) or OPT101 | 3-10 EUR |
| ADS1115 / ADS1015 breakout | 5-10 EUR |
| IR LED 860-870 nm | 1-2 EUR |
| LED constant-current source + low-noise regulator | 3-8 EUR |
| Long-pass filter 830-850 nm (AliExpress band-pass/long-pass) | 5-15 EUR |
| BNC connector, trigger interface, board, wires, supply | 10-15 EUR |

**Cheap version about 30-40 EUR.** A more refined one (OPT101, faster 16-bit ADC, branded
865 nm interference filter) about 60-100 EUR. Already owned: ESP32, triggerable flash, cuvettes.
A narrow interference filter at 865 nm is not needed with 50 mOD signals.

## Open questions

- Trigger level and polarity of the flash input (and whether it has a TTL sync output).
- Flash pulse duration.
- Absorbance of the preparation at 865 nm and the optical path (1 cm cuvette assumed).
- Which long-pass filter, and the exact LED part (check the peak wavelength on the datasheet).
- Mechanical holder for LED, cuvette, filter and photodiode (to be 3D printed, like the tower
  in `esp32-od-monitor`).

## Next steps

1. Circuit schematic: LED driver, photodiode amplifier, ADS1115, trigger interface.
2. ESP32 firmware: trigger output, pre/post-trigger burst acquisition, averaging, transmission of the trace.
3. Host side: receive traces (serial or MQTT), plot the averaged decay, exponential fit. Plots must
   use the shared PlotStyleKit style.
4. Reuse of `esp32-od-monitor` where it fits: the ESP32 toolchain and the MQTT/Influx/Grafana
   pipeline. That pipeline carries one value per second, so traces of N points per flash need
   a different payload and a trace viewer.

## Related projects

- [`esp32-od-monitor`](../esp32-od-monitor): turbidity (OD) monitor with TSL2591 sensors, source of the ESP32 toolchain and server stack.

## License

MIT, see `LICENSE`.
