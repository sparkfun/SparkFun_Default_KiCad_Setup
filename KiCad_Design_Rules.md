# KiCad Schematic & Board Rules (SparkFun style)

Caution: This document was written for machines, not humans.

## Schematic: wiring

1. **One wire segment between components, and keep it short.** Connect pin to pin (or pin to node) with a single straight segment. Don't chain collinear segments or run a "rail" past the point where it is needed. A single segment is still wrong if it is longer than it needs to be: pull the part in until the only thing setting the gap is the part's own field text (rule 3).
   - **Regulator input/output caps:** the cap node sits as close to the VIN/VOUT pin as the cap's field text allows: Ref / Value / Voltage / Tolerance must clear the IC's pins, the EN tie and the body, with ~1.27 mm to spare. For an RT9080 in SOT-23-5 that is 7.62 mm from the EN corner (the C1 node lands 8.89 mm from the VIN pin end), not a long run across the block. The output cap sits one stub (1.27 mm) from VOUT, since its text points away from the IC.
   - After pulling parts in, re-centre the block (rule 14a).

   ```
   WRONG:  VRAW                                RIGHT:  VRAW
            ●──────────────────────●─ VIN               ●──────●─ VIN
            C1 10uF                └─ EN               C1 10uF └─ EN
   ```
2. **Put power symbols and decoupling caps on the same node.** The cap sits directly under the junction where the power symbol attaches, joined to it by a short stub (rule 4). Don't run the wire past the power symbol to reach a cap further along.

   ```
   WRONG:  VOUT ──────●──────────┐        RIGHT:  VOUT ────●  (3.3V above, cap below,
                    3.3V        C2                          │   one segment from the pin)
                                                            C2
   ```
3. **Keep wires as short as possible. The stub length is 0.050" (1.27 mm, one grid step).** Every stub (pin → label, pin → power symbol, node → part, part → GND) is exactly 1.27 mm. Longer wires only where the next part's field text needs the room. **Never use two segments where one will do.** A straight run is one segment. A connection with a bend is exactly two segments meeting at the corner, with nothing else in between. Bends only when geometry forces them (e.g. a pull-up feeding an EN pin beside VIN).
4. **A pin only ever connects to a wire.** Every pin end, on ICs, connectors and passives alike, gets a wire (normally 1.27 mm / 0.050") before it meets anything else: another pin, a power symbol (GND, 3.3V, VBUS…), a global label, or a junction.
   - **Labels too:** a global label never sits on a pin end. Draw one 1.27 mm (0.050") wire segment out of the pin and put the label on its end. Only no-connect (X) flags go directly on pins.
   - A GND symbol under a resistor or cap hangs from a short wire stub, not from the pin itself.
   - Stacked parts (R → LED → jumper) have a wire between each pair.
4a. **Power symbols point up (supplies) or down (GND), each on its own 1.27 mm vertical stub.** A power symbol never sits on the end of a horizontal wire or directly on a horizontal node. From a side pin, run the 1.27 mm horizontal stub, then one 1.27 mm vertical segment up (VBUS, 3.3V) or down (GND) to the symbol. On a horizontal node (e.g. regulator output → cap), the symbol rises from the node on its own 1.27 mm vertical segment.
    - **No sideways power symbols, ever.** On a 0.1" header, route a supply pin out 1.27 mm, then run a vertical wire **up past the top of the connector** to the supply symbol. Route a GND pin out (one more grid step), then run a vertical wire **down past the bottom of the connector** to the GND symbol. Adjacent GND pins join the same vertical run.
    - Put the supply run nearer the connector and the GND run one grid further out. The other pins' net-flag wires cross these runs at right angles with **no junction**.
    - **Move net flags out to make room.** All net flags on that connector move out together to one aligned column, just beyond the outermost power run.

    ```
             ↑ 3.3V
    RAW ─────┼───── 1                 (flags in one column, left of the runs)
          ┌──┼───── 2  GND
    RST ──┼──┼───── 3
          │  └──── 4  3.3V
          │      × 5 …
    SCK ──┼──────── 9
          ⏚ GND
    ```

    ```
    WRONG:  VBUS ──┐↑          RIGHT:  VBUS ──┐
                  (symbol on wire end)        │   ← 1.27 mm vertical stub
                                              ↑ VBUS
    ```
4b. **Net flags follow the same rules as power symbols.** A global label never sits on a junction or a wire corner. Give it its own 1.27 mm segment out of the node, pointing away (e.g. `~{RESET}` hangs 1.27 mm below the R3/EN junction, and `RAW` sits 1.27 mm left of the C1 junction). A label may only sit on the free end of a wire that runs straight into it.
4c. **Parts with a vertical pin connect like power symbols.** An antenna, test point or similar single-pin symbol whose pin points down or up never sits on the end of a horizontal wire. Run the horizontal wire to the corner, then one 1.27 mm vertical segment into the pin.
5. **Pin to pin is one continuous wire.** When a pin feeds a part that isn't in line with it, draw a single wire from the pin end straight to the part's pin, with one bend at the corner. Don't break it into pin → stub → corner pieces, and don't add a separate stub at the far end.
6. **Align related parts on a common row.** Parts in the same group (CC pull-downs, I²C pull-ups, decoupling caps, π-match shunts) sit at the **same height**, so their top pins, bodies and GND symbols line up. Put the row as close as the geometry allows: the top pins sit 1.27 mm below the lowest feeding wire, and **the nearest part sits only 1.27 mm (one stub) from the source pin**. So the first horizontal run off the pin is 1.27 mm before it turns down, and the next part steps out only as far as its field text needs (≈10.16 mm for a resistor). Each connecting wire then drops straight down into its part. Place the part nearest the source first, and step the others outward so wires never cross.

   ```
   WRONG (staggered, stubs on stubs)       RIGHT (aligned, one wire each)
   CC1 ───────────────┐                    CC1 ─────────┐
   CC2 ──────┐        │                    CC2 ─┐       │      (first run = 1.27 mm)
             R2       R1                        R2      R1
             ⏚        │                         │       │
                      ⏚                         ⏚       ⏚
   ```
6a. **Text must clear the feeding wire: drop the whole group.** When parts hang from a horizontal wire with fields beside them (e.g. decoupling caps with Ref / Value / Voltage / Tolerance on the right), the top field (Reference) must sit fully below the wire. If it would touch, move the **entire group** down one grid step (1.27 mm, so the drop from the node becomes 2.54 mm). Keep all parts in the group on the same row and their GND symbols aligned. Don't move the text away from its part, and don't stagger the parts.

    ```
    WRONG:  3.3V ●──C3────────┐         RIGHT:  3.3V ●────────────┐
                 │ 2.2uF      │ C4                   │            │
                 ═            ═                      │ C3         │ C4
                                                     ═ 2.2uF      ═ 0.1uF
    ```
7. **Junction dots only where 3+ connections meet on a wire.** No junctions on bare pin-to-pin points.
8. **Wires must not cross part bodies or field text.** Avoid wire crossings, except where header power runs (rule 4a) must cross net-flag wires. Those cross at right angles with no junction dot.

## Schematic: flags (net labels)

9. Use **global labels, `input` shape, 1.27 mm text** for signal nets.
10. Put labels on a **1.27 mm (0.050") wire out of each IC/connector pin** (rule 4), pointing away from the part: right-side pins point right (angle 0), left-side pins point left (angle 180).
11. **Arduino pin nets: integer + primary function, no bank.** A net that runs from the processor to a connector (and anywhere else on the sheet) is named by its Arduino pin number, then its main extra function: `0/TX2`, `1/RX2`, `2/SDA`, `3/SCL`, `4`, `A10`, `A14/SCK`. Never include the port/bank (no `PA10`, `PB12/RX`).
    - Numbers are unique, start at 0 and run **sequentially by header position**: down the left header top to bottom (0, 1, skip GND…), then down the right header top to bottom (power pins skipped), e.g. 0–17 on a Pro Micro. The pin number is a position, not a processor port. The friendly Arduino mapping (pin → port) is done in the Arduino board files.
    - Number in the **physical order down each header**, even where the legacy board ran its labels backwards (the original Pro Micro's A3, A2, A1, A0 become A10, A11, A12, A13 from the top). Assign the processor pins to the header slots first (rule 11c), then name the nets to match. Don't let old labels or the processor's port order set the sequence.
    - Add only one primary function after the number (TX, RX, SDA, SCL, CS, SCK, PICO, POCI). Put secondary functions (ADC channel, debug) in a note.
11a. **UARTs are numbered from 1, in processor order.** With only one UART broken out, label its pins plain `TX` / `RX`. With several, the processor's UART0 becomes `TX1`/`RX1`, UART1 becomes `TX2`/`RX2`, and so on. Never use `TX0`/`RX0`. The same names go in the net names (`8/TX1`, `0/TX2`) and the board silkscreen.
11b. **Analog-capable pins carry an `A` before their Arduino number.** `A5` means Arduino pin 5, which has an ADC input. The number is still the pin's unique Arduino number, so never add a separate `A0`, `A1`… numbering that duplicates pins 0, 1…. Pins without an ADC input get no `A`. Silkscreen labels follow the net (`A10`, `A11`…); SPI/I²C pins keep their function name on the silk (`SCK`, `CS`…). Add one short note under the headers saying which pins are analog capable (e.g. `A0, A1 and A7-A12 are analog capable.`).
11c. *Soft rule:* **choose which processor pin goes to which header slot so the ratsnest is easy to route.** Do this from the actual board, with the main IC in its fixed orientation, before routing. The fixed slots come first:
    - **Fixed function slots (Pro Micro):**
      - Right side, top four: `RAW`, `GND`, `RST`, `3V3`.
      - Right side, bottom four: SPI (`SCK`, `POCI`, `PICO`, `CS`).
      - Left side, top two: a UART's `TX`/`RX`. If there are several UARTs, any one may go here (prefer the one nearest the slots, or the serial-bootloader UART). If there's a UART, it must go here.
      - Left side, 5th and 6th: I²C `SDA`/`SCL`.
      - If the processor lacks SPI, UART or I²C, those slots take any GPIO.
    - **Remaining slots:** fill with the pins that have the **shortest, least-crossing path** from their physical spot on the IC. Walk around the IC edge in order, so neighbouring IC pins land on neighbouring header slots.
    - **Too many GPIO:** route out the most valuable pins (multi-function: ADC, PWM, UART/SPI alternates, timers). Leave input-only, output-only or single-purpose pins unconnected.
    - **Too few GPIO:** the spare header slots become GND (rule 13).
    - After assigning, rename the nets sequentially (rule 11) and update the board silkscreen labels to match.
    - Non-GPIO nets keep descriptive names: `USB_D+`, `~{RESET}`, `VDCID`, `ANT`.
    - No pin-map note (Arduino pin → processor port) on the sheet: the net labels at the processor already show it, and the Arduino board files hold the mapping.
11d. **Stack duplicate pins.** When a symbol has several pins that are the same signal and must always be wired together (multiple VSS/GND pins, an exposed pad that is GND, duplicated VDD pins on the same rail), stack them at one pin location so the schematic shows one pin and one wire. Follow the library convention:
    - One pin is visible and keeps the real electrical type (e.g. `power_in`). The other pins sit at the same (x, y) and orientation, **hidden**, with type `passive`.
    - Put the stack where the single pin reads naturally: GND centred on the bottom edge, supplies on the top/left.
    - Then delete the now-redundant wires and power symbols, so one stub and one symbol serve the stack.
    - **Don't stack pins that are only usually tied together**, or that the datasheet wants decoupled or filtered separately: analog vs digital supplies (AVDD/DVDD, VDDA/VDD), analog vs digital ground (AGND/DGND) when the datasheet separates them, VBAT, VREF, I/O-bank supplies on different rails, and USB/PLL/RF supplies with their own filter. Keep each of those as its own visible pin with its own cap.
    - Example: CH32V006F8U6 pin 4 (VSS) and pin 21 (EP, VSS) are stacked; pin 4 shows.
12. Use **power symbols** (SparkFun-PowerSymbol: GND, 3.3V, VBUS, …) for rails, never global labels. **No PWR_FLAG symbols.** They add clutter without adding information. A rail that no power-output pin drives (RAW, GND, internal LDO nodes) raises `power_pin_not_driven` in ERC. Add an ERC exclusion for each one, with a comment, instead of a flag. Treat other datasheet-required power-pin conflicts the same way. Example: a CH340 V3 pin must be tied to VCC for 3.3 V operation.
    - ERC exclusions are keyed by the marker's **sheet position**. When you move a part that carries an exclusion (e.g. re-centring a block), update the coordinates in the `.kicad_pro` `erc_exclusions` list, or the error comes back.
13. Mark every unused IC pin with a **no-connect (X)**. Unused **header/connector pins go to GND** rather than NC: extra ground pins help users more than dead pins. Leave a header pin NC only when GND would be wrong there (e.g. a pin a shield may drive), and label it `NC` on the board (rule 24).

## Schematic: layout & aesthetics

14. **USLedger sheet**, split into functional blocks (Postcard layout) by **gray dashed lines**: polylines with dash stroke, colour gray (RGB 128,128,128), default width. They run edge to edge between blocks and never through a part.
14a. **Center each circuit block horizontally in its block area, under its section title.** Measure the block's full extent: symbol bodies, pins, wires, labels and the field text of its parts. Shift the whole block, snapped to the 1.27 mm grid, so that extent is centred on the area between the dashed dividers. Text that belongs to a part or group (e.g. "VDD33 decoupling", "RESET", a note above a diode) moves with the block. Section-wide notes (e.g. the π-match note) stay centred under the title. Don't leave a block hugging one divider with empty space on the other side.
15. A **3 mm section title** centred at the top of each section ("USB-C", "Power", "CH592D BLE Microcontroller", …).
16. **Notes in 1.27–1.5 mm text** inside the section they explain (boot procedure, DNP options, pin-sharing caveats).
16a. **The regulator block states the allowed input (RAW) voltage range**, e.g. `RAW input: 3.6V - 5.5V`, with a line that shows how the limits were derived.
    - **Max** = the lowest of (a) the regulator's maximum recommended input and (b) the derated voltage rating of every capacitor (and any other part) on the input node.
    - **Derating:** aim for ~50% of the cap's rating (a 20 V cap → ~10 V). Up to **80%** is acceptable, and below that no change is needed. Example: a 10 V cap on a 5.5 V-max regulator input runs at 55%, which is OK.
    - **Min** = output voltage + regulator dropout at full load.
    - Only if the max input would push a cap **above 80%** of its rating (or the range falls below the expected supply, e.g. 5 V USB), change the cap to a higher-voltage part rather than documenting a range that doesn't work.
16b. **Encumbered pins get a "Pin cautions" note next to the processor block**, written as *if you do X, Y happens*. Cover any pin that has a side effect or an extra load:
    - boot/strap pins (e.g. "pin 6 held low at power-up → bootloader starts instead of your code");
    - pins with on-board pull-ups/pull-downs, LEDs or buttons;
    - pins shared with debug, reset or bootloader functions.
    - Naming a pin's function ("PB22 = BOOT") isn't enough. Say what happens to the user.
17. Field placement (relative to the symbol):
    - Vertical caps: Ref / Value / Voltage / Tolerance stacked on the right, left-justified, 2.54 mm pitch.
    - Vertical resistors and LEDs: Ref / Value on the right.
    - ICs: Ref / Value above the body.
    - Shunt caps under a horizontal signal line: fields on the left so they don't collide with the series parts.
17a. **Solder jumpers show their value, not their reference.** Hide the jumper's Reference (JP1, JP2…) and show its **Value** in the Reference's spot. The value is a short, readable function name of at most 4 characters, e.g. `PWR` (LED), `SHLD` (USB shield), `I2C` (pull-ups), `VE` (enable).
    - On the board, that same value is the jumper's silkscreen label, as a **KiBuzzard label** beside the jumper. The footprint's Value and Reference stay hidden; no plain KiCad text.
17b. **RF paths offer both an on-board and an external antenna.** Any radio output gets an on-board antenna (chip, PCB trace, SMA, etc.) **and** a U.FL connector for an external antenna. Select between them with one `SparkFun-Jumper:SolderJumper_3_Bridged12_RF` (footprint `Jumper_3_NC-1_Trace_RF`), not with 0 Ω resistors.
    - Wire the common pin 2 to the matching network. Pin 1 (bridged by a trace by default) goes to the on-board antenna. Pin 3 goes to the U.FL.
    - The user can cut the trace and solder pads 2–3 to switch to the U.FL.
    - Value is a short name such as `ANT`, per rule 17a. Add a note in the RF block that explains the default and how to switch.
    - On the board, keep the jumper on the RF trace, between the matching network and the antenna feed, with short 50 Ω runs to both branches.
17c. **Every impedance-controlled trace has its width calculation on the schematic.** This covers RF/antenna feeds (50 Ω), USB (90 Ω differential), Ethernet and the like. Place a note in the block that owns the trace, in the mosaic-G5 format:
    - the trace type and target impedance (e.g. "Microstrip (Coplanar Single Ended) 50Ω");
    - the calculator used (e.g. https://jlcpcb.com/pcb-impedance-calculator);
    - the **full fab stackup name** (e.g. `JLC04081H-7628`), board thickness, layer count and Er;
    - the dielectric thickness to the reference plane, the copper thickness and the reference plane (e.g. In1.Cu GND);
    - the polygon isolation (gap) and the resulting **trace width** (and spacing for diff pairs), in mil and mm.
    - Use the stackup the board will actually be ordered with: the same name as in the PCB's Ordering Notes (rule 24f), at the real board thickness. Recalculate whenever the thickness or stackup changes.
18. **DNP parts** are flagged `dnp` (KiCad draws the red X); value "DNP" for unfitted caps.
19. Bottom-right: SparkFun logo, OSHW logo, Qwiic logo (if Qwiic), fiducials (rule 19a), in the same positions as the Postcard. Title block has title, date, rev and "Designed by" in comment 1.
19a. **Fiducials only on sides that have assembled parts.** Fiducials exist for the pick-and-place camera, so a side with no placed components gets none. A single-sided board (rule 24a) has fiducials on the top only. Pads, solder jumpers and artwork on the back don't count as parts. Use the schematic symbol count to match: one FID symbol per fiducial actually placed.
    - **Templates and copied boards bring 4 fiducials** (2 top, 2 bottom). When a board starts from the Postcard, an older board or a template, check the back straight away: if it has no assembled parts, delete the bottom FID footprints **and** their FID schematic symbols.
    - The check is mechanical: list B.Cu footprints and ignore `board_only` items, jumpers, solder pads and logos. If nothing is left, there must be no FID on B.Cu (rule 28).

## Parts & libraries

19b. **Version numbers have a period: `v1.0`, never `v10` or `v01`.** Write every revision as `v<major>.<minor>` (`v1.0`, `v1.1`, `v2.0`). This covers the schematic and PCB title-block `rev` fields, the board's KiBuzzard revision mark, file names and release notes. All of them carry the same string. **Every new board starts at `v1.0`**, never `v0.1` or `v0.x`. Later spins count up from there (`v1.1` for a small fix, `v2.0` for a major change).
20. Use **SparkFun KiCad library** parts (`SparkFun-*`) wherever one exists. They carry footprint, PROD_ID, voltage/tolerance fields.
20a. **Every purchased component has a fully formed PROD_ID** (e.g. `CAP-14259`, `IC-22431`). An empty field or a placeholder (`XTAL-`, `IC-...`) is not done.
    - If a part has no PROD_ID, **tell the user**: they need to either pick an existing stocked part or add the new part to the internal parts database and fill in the number.
    - Items that aren't purchased parts (castellations/headers that are only copper, solder pads, solder jumpers, fiducials, logos) are set **off the BOM** (`in_bom no`) instead, so they don't need a PROD_ID. DNP parts don't need one either.
20b. **Review the feeder count before release.** Count the pick-and-place feeders: one per unique PROD_ID among placed BOM parts. Then give the user a list of consolidations, each with the reason it's safe:
    - Same value on different packages/ratings → one part (e.g. two 10 µF caps → both 0603 10 V).
    - Values that only need to be "about right" → move them to a value already on the board: decoupling/bulk caps within the IC's allowed range, LED resistors, pull-ups, button series resistors.
    - Use the larger package (0603) where there's room and it matches an existing feeder, and prefer values SparkFun already stocks.
    - Never merge values that set function: USB-C CC 5.1 k, crystal load, RF matching, regulator feedback, values the datasheet requires.
    - Report the before/after feeder count. The user decides; don't change parts without asking.
20c. **Prefer multi-source parts.** For anything that isn't a unique IC (antennas, crystals, connectors, passives), pick a part whose package and footprint are made by **at least 3 manufacturers** with good stock, so that no single supplier can stop production. Note the alternates in the part's description or in a schematic note.
21. **No project libraries.** A part missing from the SparkFun library is created **in the SparkFun KiCad library**, in the matching sub-library (MCUs → `SparkFun-IC-Microcontroller`, QFN/SOT packages → `SparkFun-Semiconductor-Standard`, connectors → `SparkFun-Connector`, RF → `SparkFun-RF`…), and the design uses it from there. The project keeps no `sym-lib-table` / `fp-lib-table` of its own.
    - Before adding a part, check whether the library already has it, comparing pad geometry and not just names. Reuse an identical footprint rather than adding a renamed duplicate.
    - 3D models go in the library's `3dmodels/<Lib>.3dshapes` folder and are referenced as `${SPARKFUN_KICAD_LIBRARY}/3dmodels/...`. Never point to `${KICAD10_3DMODEL_DIR}`, and check that the STEP file actually exists.
    - **If KiCad's library has a good, compatible 3D model, copy it into the SparkFun library** (same file name, matching `.3dshapes` folder) and point the footprint at the SparkFun copy. Compatible means the same body size, pitch, pin count and pin-1 orientation, with the exposed pad within ~0.1 mm of the land. Compare against the footprint's pads, not just the name: the closest KiCad model for a WCH QFN-20 3×3 P0.4 is `UQFN-20-1EP_3x3mm_P0.4mm_EP1.85x1.85mm.step`. Check the SparkFun folder first (compare checksums), and reuse an identical file rather than adding a renamed copy. KiCad models are licence-clean (rule 24b).
    - Every footprint has a courtyard (F.CrtYd, 0.25 mm clear of the pads).
    - Build new symbols in SparkFun style:
        - 2.54 mm pin length, 1.27 mm text.
        - Power pins on the left/top, GND at the bottom, I/O on the right.
        - Short alternate-function pin names.
22. Footprints: SparkFun land patterns, Reference on F.Fab (0.5 mm), courtyard on F.CrtYd, pin-1 dot on silk.

## Board

23. Keep the board **linked to the schematic** (footprint path = symbol UUID). DRC with schematic parity must show 0 issues.
24. **All silkscreen text is KiBuzzard labels** (font FreddySpark-Regular, no caps unless it's a pin tag), never plain KiCad text. This covers pin labels, part/jumper/button labels, USB pad labels, the product name and the revision mark. Height ≥ 0.8 mm.
    - Vertical pin labels between 2.54 mm castellations: ≤ 3 characters on top; full names on the back where there is room.
    - *General guideline (not a hard rule):* **top-side header labels are pin numbers only**, i.e. the Arduino number with `A` for analog pins (`0`, `7`, `A10`, `A14`). Don't put function names (`TX`, `SDA`, `SCK`, `CI`) on top; they go on the back. Power and control pins (`RAW`, `3V3`, `GND`, `RST`) keep their names. Break the guideline only when a number alone would genuinely mislead.
    - Every button gets a label (e.g. `R` / `B`), and every header pin gets a label on both sides.
    - A header pin that stays NC is labelled `NC` (KiBuzzard) on both sides. Never leave an old net or function name on it.
    - Remove leftover silk from imported designs (old part outlines, loose lines, diagram art) that no longer matches a part.
    - **Pin labels sit 1.2 mm from the pin, top and bottom.** Measure from the pad centre to the label's nearest ink edge. This applies to vertical top labels (the baseline faces the pad) and horizontal back labels alike, so short and long names line up along the header. Centre each label on its pin row. Never centre a label on an old text origin, or the gap will vary with word length. Only a label on an end pin may slide outward along the header, and only if it would come within 0.5 mm of its neighbour.
24a. **Single-sided assembly.** Every assembled part goes on the top side. Only copper-only items may go on the back: solder pads (e.g. USB pads), **solder/cut-trace jumpers (PWR, SHLD…) with their KiBuzzard labels** and artwork. No fiducials on the back (rule 19a). Place parts near the pins they serve, with no new courtyard overlaps.
24b. **3D models must be licence-clean.** Use the KiCad library models (CC-BY-SA 4.0 with the design exception) or SparkFun's. Don't use vendor/EasyEDA models whose licence isn't clearly free, and never models with branding on the part.
24c. **Qwiic logo next to the Qwiic connector**, on the same side as the connector. Use the SparkFun-Aesthetic `qwiic_Nmm` artwork at the largest size that clears pads and courtyards. If the schematic's qwiic_Logo symbol is used elsewhere, add the extra logo as a board-only footprint.
24d. **Spare GPIO gets one addressable LED.** If the micro has a pin no header, connector or function uses, add a single one-wire pixel (e.g. WS2812B-2020 / SK6805-EC20) on that pin, with its 0.1 µF decoupling cap, and label the net with its Arduino number + `/LED`. If every GPIO is used, leave it off. (Pro Micro CH592F: all 20 GPIO are used, so it has no LED.)
24e. **Centre the design on the drawing sheet.** Move the whole board (outline, parts, zones, graphics) so the board outline sits in the middle of the sheet. Keep the shift on the 1.27 mm grid, and put the grid/aux origin on the board's lower-left corner. Staged off-board parts move with it.
24f. **Ordering Notes in the title block.** Put the fab notes on **Eco1.User** in the open lower-left part of the title block, as on the mosaic-G5 boards (A4: heading "Ordering Notes" 2 mm at 199.39, 168.91; two columns of 1.27 mm text at x 205.74 and 236.22, 1.905 mm line pitch from y 172.085). List solder mask, silkscreen colour, layers, thickness and finish in column 1, and stackup, material, special features (e.g. castellated holes) and outer/inner copper weights in column 2. Take the values from the design, not the template. Fill the board's title block (title, date, rev from the schematic; company "SparkFun Electronics"; comment 1 "Designed by: …").
24g. **The OSHW logo brings its licence.** Whenever the board carries the OSHW logo, place `SparkFun-Aesthetic:Creative_Commons_License` (CC BY-SA 4.0 text on Eco2.User) just above the title block (A4: 232.41, 159.385), as a board-only footprint if it has no schematic symbol.
24h. **Ask about battery charging + JST.** At the start of every board, ask the user whether to add a LiPo charger (e.g. MCP73831) with a 2-pin JST-PH battery connector. Don't add it without asking. If the board has no room (e.g. Pro Micro CH592F), still remind the user that it was left off and why, so the choice is deliberate.
24i. **On-grid placement: new parts sit on the 12.5 mil grid.** Every part you add or move has its footprint origin (the centroid for SparkFun footprints) on a 12.5 mil (0.3175 mm) grid, measured from the grid origin (rule 24e; keep that origin on the grid too). Place with the 12.5 mil grid active, and snap any part you nudge back onto it. Example: when the CH340E was added, its centre should land on a grid point, not at an arbitrary spot.
    - Rotation doesn't change this: the origin stays on grid at any angle.
    - **Exclusions:** parts whose position is set by something else may stay off grid:
        - mechanically fixed parts (castellated headers, the USB-C connector, mounting holes);
        - parts imported from a reference design and left where they were;
        - RF/impedance parts placed to suit the trace geometry;
        - a part that has to sit tight against another part's pads;
        - artwork and KiBuzzard labels, which are placed by rule 24.
    - Don't move existing off-grid parts just to tidy them unless asked. The rule applies to parts you add or move.
25. **Castellated half-holes** are exempted by a project `.kicad_dru` rule (edge clearance ignored, annular ring 0), not by editing library footprints.
26. Keep the **GND / 3.3V planes** as zones. Remove dead-net zones and leftover routing when the design changes.
26a. *Soft rule:* **on a simple board, pours sit exactly on the outline.** When the board outline is a plain rectangle (or another simple shape on the grid), draw each full-board pour (GND, 3.3V…) with its corners on the Edge.Cuts corners, i.e. on the 12.5 mil grid, not a hair outside the edge (e.g. 139.7 / 157.48 mm, not 139.6492 / 157.5308 mm). The zone's own edge clearance keeps copper off the edge. Doesn't apply to complex outlines (cut-outs, curves, odd shapes), where the pour follows whatever geometry works.
27. Back-side footprints are mirrored KiCad-style (y negated, angles negated, layers swapped, text mirrored with left/right justify swapped). Use only library footprints so "footprint doesn't match library" stays clean.
28. **Verify** before calling it done:
    - `kicad-cli sch erc`: 0 errors (documented exclusions per rule 12 don't count).
    - Netlist matches the intended connections.
    - `kicad-cli pcb drc --schematic-parity`: only unrouted / known-inherited items.
    - Audit: no wire chains, no over-long single segments (rule 1), no pin touching a pin, power symbol or label directly, duplicate pins stacked (rule 11d).
    - On grid: every footprint origin is on the 12.5 mil grid from the grid origin, except the rule 24i exclusions.
    - Fiducials: no FID on a side without assembled parts, and the FID symbol count matches the placed fiducials (rule 19a).
    - Every footprint's 3D model resolves to `${SPARKFUN_KICAD_LIBRARY}` and the file exists (rule 21).
    - Render the schematic and both board sides and inspect them.
