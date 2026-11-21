# CW312T ESP32-S3, rev 01

This 3.3 V CW312 card uses the Espressif ESP32-S3-WROOM-1-N16R8. The module has 16 MB of flash, 8 MB of octal PSRAM, and a PCB antenna. Open the KiCad 10 project to edit the schematic and PCB. The BOM and CW312 contact list are in `bom.csv` and `cw312-pinout.csv`.

## Power and current

J1 is the only power input. Connect regulated 3.3 V to J1.1 and ground to J1.2. Use a supply that can deliver at least 1 A for bench tests. Keep voltage at TP2 within 3.0 to 3.6 V, including during startup and radio activity.

The CW313 passes the capture unit's 3.3 V supply to A5. Husky specifies 200 mA at its target connector. The S3 module datasheet lists up to 355 mA peak in its Wi-Fi transmit test. A first-order charging estimate for 22 µF of input capacitance is 73 mA for a 1 ms, 3.3 V ramp. A faster ramp can exceed 200 mA before the device starts. The card leaves A5, the other CW313 supply pins, and both SMA shunt pins open. There is no path from J1 to those supplies. A26 supplies the debugger's voltage-sense input through R5. Use a debugger that takes a voltage reference at this pin. Leave B10 open.

R1 is the fitted 0 Ω series link. Remove R1 with power off and connect an ammeter between TP1 (source side) and TP2 (load side). The meter includes the module, its decoupling, bias resistors, and any current drawn at the debugger reference. It excludes the bench supply's own current. Check the voltage at TP2 under load. Restore R1 after the test.

Signal pins connect directly to the module. Before energizing the card, power the CW313 host and set its connected I/O to high impedance. Enable host and probe drivers only after the target rail is up. Put the drivers back in high impedance or disconnect the cable before removing target power. This procedure prevents current through signal protection diodes when one side is off.

## Bring-up

1. Keep all power off. Align the key with the CW313 slot. The module side faces the CW313 component side. Check that the card is seated and the antenna projects away from the baseboard.
2. Connect bench power at J1. Check TP1, TP2, and TP3. Confirm that EN at TP4 rises after the supply. Press RESET and confirm that TP4 goes low.
3. Connect a 3.3 V UART adapter to J2. Use J2.1 for ground, J2.2 for target TX, and J2.3 for target RX. TP5 and TP6 expose those signals. When the adapter drives RX, keep Husky's UART TX at high impedance. Hold BOOT, tap RESET, and release BOOT to enter ROM download. Use Espressif esptool with the ESP32-S3 target. Tap RESET with BOOT released to run the image.
4. For USB Serial/JTAG, solder a short USB full-speed data cable directly to J3. J3.1 is ground, J3.2 is D−, and J3.3 is D+. Insulate the cable's VBUS wire. Power J1 before attaching the cable to the host. Use the ESP32-S3 built-in USB-JTAG OpenOCD configuration. The optional pin header is DNP.
5. For external pad JTAG, the four signals are on the CW313 JTAG header. Use the target-side reference on A26 and local RESET. The module defaults to USB Serial/JTAG. GPIO3 is pulled low for the external pad selection if the corresponding strap-selection eFuse is enabled. Check the device eFuse state before selecting this debug path. The fitted GPIO3 resistor also gives the strap a defined state. GPIO46 and GPIO45 use their internal defaults. IO35 through IO37 are reserved by the module's PSRAM.
6. Configure GPIO4 as an output for the trigger on CW312 B19. Set CW313 JP2 to GPIO4. Keep the baseboard end configured as an input.

## Mechanical and fabrication checks

The CW312 finger sides, 1.00 mm pitch, 3.00 mm pitch across the key, 0.70 × 4.20 mm lands, and 34.30 mm tongue span follow the template and match the RP2350 contact pattern. The 1.60 mm card thickness matches the existing card fabrication notes and the Samtec -02 slot. The insertion tip chamfers are 0.50 mm in the board plane. Fit the card like the RP2350 reference, with pin B1 on the component side. This is the CW312 pinout.

The template outline is 58.675 × 53.691 mm. The far edge has a 1 mm recess to let the complete 6 mm antenna region overhang. The assembly has no card substrate under the antenna, and an all-copper keepout extends around it. Other components start above the slot contact area. In the nominal mated position, the antenna is more than 40 mm from the slot and farther from the CW313 copper. Keep cables, fixtures, and metal at least 15 mm away from the antenna. Check this clearance on the actual bench assembly.

Use a four-layer 1.60 mm board with continuous ground copper around the module and beneath the USB pair. Use 0.15 mm minimum trace and clearance, 0.25 mm minimum copper-to-edge clearance, and 0.20 mm minimum plated drill. Use a process that fills and caps the small vias in U1's exposed pad to prevent solder wicking. Specify a flat gold finish on the card contacts. The RP2350 reference uses immersion gold with no thickness bevel. Confirm the slot manufacturer's finish and bevel requirements if ordering for repeated mating. The PCB has no paste on the edge contacts.

KiCad 10.0.6 ERC reports zero violations. DRC reports zero errors, zero unrouted items, and zero schematic parity issues. The one warning records the deliberate removal of the two U1 silkscreen strokes that would extend beyond the PCB. Pad, paste, drill, and courtyard geometry are unchanged. The Gerbers and Excellon files represent rev 01. Qualify slot insertion, probe access, USB operation, startup and RF current, and antenna performance on a first article.

## References

- NewAE CW312 template rev 0, `cw312_template.PcbDoc` and `cw312_template_cardedge.SchDoc`, commit `40c337d2c8c15345bab03c9aa2072a23a36ccc3b`. The template license field says TBD.
- NewAE CW312T_RP2350 rev 02, schematic, PCB, fabrication notes, and insertion photo. The project is marked NDA. The KiCad project has newly drawn card geometry. The original Altium files served as dimensional and connectivity references.
- NewAE CW313 rev 02 schematic and PCB, commit `7b689277ca6852fccd195c29525a26640b6296d0`. The schematic title block states CC SA 3.0. [CW313 design files](https://github.com/newaetech/chipwhisperer-target-cw313/tree/7b689277ca6852fccd195c29525a26640b6296d0)
- Samtec PCIE-XXX-02-X-D-RA series drawing, sheet 1 rev C and sheet 2 rev D. [Connector drawing](https://suddendocs.samtec.com/prints/pcie-xxx-02-x-d-ra-mkt.pdf)
- Espressif ESP32-S3-WROOM-1/WROOM-1U datasheet v1.8, sections 3, 4, 6, 9, and 11. The standard KiCad land pattern was checked against figure 11-1. [Module datasheet](https://documentation.espressif.com/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf)
- Espressif ESP32-S3-DevKitC-1 schematic v1.1, 2022-04-13. Review the reset RC, boot switch, and UART circuit. [DevKit schematic](https://dl.espressif.com/dl/schematics/SCH_ESP32-S3-DevKitC-1_V1.1_20220413.pdf)
- NewAE Husky target connector power limit. [Husky hardware manual](https://rtfm.newae.com/Capture/ChipWhisperer-Husky/)
