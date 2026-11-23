# Anet A8 Vaarai — configuration and maintenance

## Current status

- Source configuration defines `NOZZLE_TO_PROBE_OFFSET { 0, -29, -2.06 }`.
- The last known installed firmware is the Marlin 2.1.2.9 build of November 9,
  2026 at 21:43:22, with EEPROM enabled and compiled Z−2.00 default. The source
  change to Z−2.06 has not been uploaded.
- The user tested and saved `M851 Z-2.06` in EEPROM; readback after restart has
  not been reported. EEPROM values can differ from compiled defaults: do not
  reset without preserving calibration.
- 2×2 bilinear leveling and G34 are enabled. The saved mesh was recovered after
  restart; G28 disables compensation, which can be reactivated with `M420 S1`
  after checking for a valid mesh.

The dated paragraphs below describe historical stages.

**Historical:** **2×2** bilinear leveling, 40 mm margin, uploaded and verified
(Marlin 2.1.2.9 `Nov 9 2026 18:56:24`, flash 89984/RAM 3443). Probe points
X40/180 × Y40/180; nozzle Y69/209 given offset −29. No G29 or movement executed
by the assistant. M503 reported M420 S0 Z10, M851 X0/Y−29/Z−2. EEPROM disabled;
G28 disabled the mesh and Orca startup did not run G29. First test and rear-clip
clearance required user validation.

**Historical:** user-validated Z−2.00 recorded in `Configuration.h`. Build
80500 bytes / RAM 3332 uploaded and verified on COM5; M115
`Nov 9 2026 18:04:38`, M503 confirmed M851 X0/Y−29/Z−2.00. EEPROM disabled;
the value persisted in firmware.

**Historical:** firmware uploaded `Nov 8 2026 21:24:26`, XY origin intended at
the glass front-left corner, X_MIN−11/X_MAX242, Y_MIN−11/Y_MAX219; probe at
center 110/110, nozzle 110/139. M851 X0/Y−29/Z−2.15 read back. Orca profile
offset 0x0, area X0..220/Y8..212 for clips. Reference frame and Z at the new
point remained to be checked by the user.

## Reference branch

- Machine branch: **`anet-a8-vaarai`**, published to `origin` (fork
  `https://github.com/Vaarai/Marlin.git`).
- Official repository: `upstream`, `https://github.com/MarlinFirmware/Marlin.git`.
- Integrated stable base: **Marlin 2.1.2.9**, official tag, commit `7d300b42ac`.
- Machine settings: `Marlin/Configuration.h` and `Marlin/Configuration_adv.h`.
  These versioned files are authoritative; do not replace them with examples or
  defaults during an update.

The machine branch carries customizations on top of official releases. Each
future change should be an explicit commit describing its purpose and validation.
`.pio/` outputs are not versioned.

## Hardware and features

**Historical November 8, 2026 update:** `M851 X0/Y−29/Z−2.15` was installed and
read back after flashing (Marlin 2.1.2.9 `Nov 8 2026 19:22:05`, 80500 bytes
verified). The user validated Z−2.15 with paper at the previous point; checking
the new safe point remained. Corrected Orca origin was **−11x−17** and the then
safe point was probe 121/117, nozzle 121/146. Dated build and coordinate entries
below describe earlier stages.

| Item | Configuration |
|---|---|
| Machine | Modified Anet A8, **MKS Gen L V2.1**, `BOARD_MKS_GEN_L_V21`, integrated ATmega2560 |
| Drivers | DRV8825 X/Y/Z/Z2/E0 |
| Dual Z | Two independent drivers; Z2 on E1, STEP36/DIR34/ENABLE30 |
| Thermistors | Type 5 for hotend and bed |
| Firmware geometry | Bed 220×220; current XY minima −11/−11, maxima 242/219; Z 0..230 |
| X/Y/Z/E steps/mm | 100 / 100 / 400 / 111.112 |
| Maximum speeds | 400 / 400 / 8 / 50 mm/s |
| Maximum accelerations | 500 / 500 / 200 / 1000 mm/s² |
| P/R/T accelerations | 500 / 1000 / 500 mm/s² |
| Motion | S-curve, Junction Deviation 0.02; G2/G3 arcs |
| Linear Advance | Enabled; firmware K0; K0.06 supplied by the OrcaSlicer PLA profile |
| Control | Hotend PID 22.20 / 1.08 / 114; Kc100; bang-bang bed |
| Protections | Thermal, watchdog, emergency parser; minimum extrusion temperature 160 °C |
| Communication | Serial 0, 250000 baud; host actions/prompts/notifications |
| EEPROM | Enabled; settings persist with M500 |
| Probe and Z safety | 3D Touch D11/D19 Z homing and Z_SAFE_HOMING installed; Z−2.06 saved per user; NC chain to implement |

The existing Z switch was reported unreliable and still needs repair. Standard
`KILL_PIN` is not a validated fast anti-collision stop.

The user clarified the hardware after the first commit: the old
`BOARD_RAMPS_13_EFB` declaration was replaced with the dedicated MKS target.
Active function pins match, but servo pins do not: SERVO0 is **D11** on this
board, SERVO1 **D21**. D7 drives HE1 and must not be used for a probe configured
as on RAMPS 1.3.

Historical probe wiring: control/power on **J24 D11/5V/GND**, sensing on
**Z+ J8 S(D19)/GND**, preserving Z− D18 for the endstop or future NC chain.
BLTOUCH and probe homing enabled in source; NC monitoring remains to be done.
At that stage probe offsets were **`{0,-29,0}`**: X0 and Y−29 confirmed, Z still
to calibrate. This configuration was uploaded and verified November 7, 2026;
M503 confirmed **M851 X0/Y−29/Z0**.

The historical remeasured Orca origin correction **X11/Y−17** was distinct from
probe offsets and remained in the slicer profile; do not confuse them or combine
them with M206 without reexamining the machine coordinate system.

User tests: `M280 P0 S10` deploys the pin, `M280 P0 S90` retracts it, `M119`
reads `z_probe`. Switch mode `M280 P0 S60`, if supported, makes triggering
easier to read; wait one second between commands. `M401/M402` are unsuitable
for a strictly no-axis-movement test: Marlin's deployment path can clear Z.
Tests performed by the assistant must remain probe-only, as authorized below.

## Offline build

From the repository root with PlatformIO Core installed:

```sh
pio run -e mega2560
```

For migration validation, first clean:

```sh
pio run -e mega2560 -t clean
pio run -e mega2560
```

Outputs: `.pio/build/mega2560/firmware.hex` and `firmware.elf`.

October 26, 2026 validation: clean build succeeded on Atmel AVR 4.0.1, Arduino
AVR 5.1.0, AVR GCC 7.3.0; flash **66250/253952**, RAM **3198/8192**. Only
warning: automatic Z2 assignment to free E1 port, consistent with confirmed
wiring. Configuration unchanged from local 2.1.2.7; no feature adaptation
required for 2.1.2.9.

After correcting the MKS Gen L V2.1 target, the build succeeded with the same
flash/RAM sizes. AVR preprocessor comparison of active pins showed motors,
minimum endstops, thermistors, hotend, bed and fan unchanged. The board fix was
a local change after the initial commit at that stage.

November 5, 2026 test build, BLTOUCH, M280 and native `PINS_DEBUGGING`:
**flash 79800/253952, RAM 3320/8192**. Preprocessor check: servo0=D11,
probe=D19, Z−=D18, probe pull-up enabled, probe homing disabled.

**Installed and verified November 5, 2026 on COM5:** Marlin 2.1.2.9, build
`Nov 5 2026 22:49:53`, 79800 bytes written and verified by avrdude, ATmega2560
signature `0x1e9801`. M115 reported Z_PROBE=1 and M503 retained main settings.
USB-only maintenance without 12 V per user context.

M43 S measured **8 ms ±4 ms** contact pulse, compatible with pre-V3.1 BLTouch.
M119 reactivates switch mode in this version: deployed `open`, retracted
`TRIGGERED` on read. M119 alone missed the brief contact. Pin commanded
retracted at end. After initial irregularities, five M43 S tests gave **5/5
contacts**, widths 10/8/10/12/12 ms (±4 ms), and the user confirmed **five clean
retractions** on first contact without assistance or spurious movement. Height
repeatability and homing/probing remained to be validated. No XYZ/E movement or
heating performed.

**Configuration only:** no changes under `Marlin/src/`. M280/M119/M43 S are
Marlin functions; only PINS_DEBUGGING was uncommented to make M43 S available.
Do not use M43 T in these probe tests.

Configuration installed November 7, 2026: offsets **X0/Y−29/provisional Z0**,
probe Z homing and **Z_SAFE_HOMING** enabled. Probe center in firmware
coordinates **99/117**, nozzle **99/146**. Calculation: Orca 220×200 usable
area center (110,100), minus Orca correction (11,−17), then minus probe offset
(0,−29) for the nozzle. Offline coordinate and travel checks passed. Z approach
2 then 1 mm/s. Build **flash 80452/253952, RAM 3320/8192**, 80452 bytes written
and verified on COM5, signature `0x1e9801`. M115: Marlin 2.1.2.9,
`Nov 7 2026 15:32:26`. M503 confirmed M851 0/−29/0 and M206 0/0/0; main settings
retained. No homing or axis movement by the assistant. Pin retracted, port closed.
At that stage Z had to be calibrated before printing; M500 could not save it,
so it had to be entered in source after validation. The user later confirmed
homing worked; Z-offset still needed measurement at that point.

## Integrate a future stable release

1. Start from `anet-a8-vaarai`, check `git status` and synchronize with the fork
   using `git pull --ff-only`. The tree must be clean before integration.
2. Check the latest **official stable release**, its notes and tag; do not
   automatically take a development branch or moving tag.
3. Fetch the exact tag: `git fetch upstream tag <version>`.
4. Examine `git diff HEAD <version> -- Marlin/Configuration.h
   Marlin/Configuration_adv.h`, format changes, migration notes, MKS Gen L V2.1
   pins and their RAMPS base, AVR HAL and active features listed above.
5. Merge the tag into the machine branch using `git merge <version>`. Resolve
   conflicts preserving machine-configuration intent and explicitly adapt renamed
   or changed parameters. Do not resolve everything using “ours” or “theirs”.
6. Check configuration diff and effective macros, particularly polarities/motor
   direction, Z2 pins, thermistors, geometry, limits, Linear Advance, S-curve,
   JD and protections; build cleanly for `mega2560`.
7. Update this document with the integrated base, adaptations and build result;
   create a commit for additional adaptations if needed.
8. Push using `git push origin anet-a8-vaarai`. Preserve published history:
   integrate future stable releases by merge, without rebasing or force-pushing.

## Printer operation

Normal connection: **Raspberry Pi / OctoPrint**. The controller is connected to
the **PC only for maintenance, without 12 V power**. Retain that context when a
PC connection is reported unless contradicted. USB 5 V can still power logic
and probe; the pin must not be assumed inactive.

XYZ movement, homing, axis-motion probing, physical extrusion/retraction,
heating, autotuning and starting/resuming prints are performed exclusively by
the user. The assistant does not trigger these actions or transmit print G-code.

User clarification: **the assistant may operate the 3D Touch alone**, without
XYZ/E movement or heating. Use direct servo/state commands; M401/M402 are not
equivalent because their routine can clear Z.

After installation by the user, `M115`, `M503`, `M119` and `M900` readings can
compare version, settings, input states and Linear Advance availability before
the user's physical tests.
