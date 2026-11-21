# CW312T STM32N657, rev 01

This six-layer CW312 card uses the STM32N657X0H3Q in the 264-ball VFBGA. It has SWD, USART1, and a 512-Mbit external serial NOR flash. Open the KiCad 10 project to edit the schematic and PCB. The BOM and all 64 CW312 contacts are in `bom.csv` and `cw312-pinout.csv`.

**Fabrication hold.** Complete the electrical and layout checks below before release.

## Power

Use a sequenced bench supply at J1.

| Pin | Supply |
| --- | --- |
| 1 | Ground |
| 2 | 3.3 V for VDD, VBAT, and the debug voltage reference |
| 3 | 1.8 V for VDDA18AON, PDR_ON, and VDDA18PLL |
| 4 | 1.8 V input for the switched auxiliary rail |

The 3.3 V and 1.8 V main domains must meet AN5967 rev 6 section 3.5. Use the supply's tracking or trigger function and verify the ramps at TP7 and TP8. During power-up, the AON threshold crossing can follow the VDD threshold crossing by up to 250 µs. While either rail exceeds 0.6 V, keep AON within 50 mV above VDD. Apply the main group before the auxiliary group.

J5 pins 1–2 connect the MCU's PWR_ON to the auxiliary switch. PWR_ON goes high after the main group reaches its thresholds. U2 then provides the 1.8 V auxiliary rail for VDDSMPS, VDDA18PMU through FB1, VDDIO3, and the flash. The internal SMPS produces the core rail through L1. The SMPS uses local input capacitors, the output capacitor bank, and the R4/C5 snubber from the AN5967 topology. V08CAP has a dedicated capacitor. USB and ADC supplies and unused VDDIO2, VDDIO4, and VDDIO5 are tied to ground. CSI supplies are grounded and CSI signals are open.

For controlled shutdown, set J5 to OFF by removing the 1–2 shunt or installing it on 2–3. R2 holds the switch off with the shunt removed. U2 discharges the auxiliary rail through QOD. The AON-powered U4 sinks current through R9 to discharge the core rail. Keep the main rails within range until TP9 and TP10 are below 0.6 V. Then lower VDD and AON under tracking control as specified in AN5967 section 3.5.2. Verify the full sequence for controlled shutdown and loss of input before release.

CW313 passes Husky's 200 mA target supply to A5. ST's AN6000 tests the core converter with load steps to 700 mA. At 0.81 V, that step is 0.57 W. At 85% efficiency it would draw about 203 mA from 3.3 V before adding any I/O, memory, or startup load. This is a sizing case. Measure the actual current at startup and under load. The card leaves CW313 supply and shunt inputs open. A26 is the target's debug voltage reference. Use a probe with a voltage-sense input. Power down or disconnect external signal drivers before switching off the target. Set CW313 I/O to high impedance until the target rail is up.

R1 is the removable 0 Ω link in the 1.8 V auxiliary input. With power off, remove R1 and connect an ammeter between TP1 and TP2. This measures U2 input current, including the internal core converter, VDDIO3, and external flash. It excludes the 3.3 V and 1.8 V main rails. Monitor TP9 as you measure. The switch and meter both cause voltage drop.

## Bring-up

1. Keep all power off. Align the key with the CW313 slot. The MCU side faces the CW313 component side. Seat the card with the same orientation as the RP2350 reference.
2. Set J5 to OFF. Apply the main rails and confirm TP7, TP8, and PWR_ON at TP11. Set J5 to RUN with the 1–2 shunt. Check TP9 for the auxiliary rail and TP10 for the core rail. Check NRST at TP4. TP3 is ground.
3. For development boot, fit a shunt across J4 pins 3–4 to raise BOOT1 and leave pins 1–2 open. Reset the target. Connect ST-LINK with SWD. J2 pins 1 through 5 are VTREF, GND, SWDIO, SWCLK, and NRST. TP5 and TP6 expose SWDIO and SWCLK. The same debug signals are on CW313's Cortex connector. Use one probe.
4. For serial boot, remove the BOOT1 shunt. Fit a shunt across J4 pins 1–2 to raise BOOT0. Reset the target. Use the 3.3 V USART1 interface on J3. J3.1 is ground, J3.2 is target TX, and J3.3 is target RX. Keep the Husky TX output at high impedance if an external adapter drives RX. ST lists PE5 and PE6 as ROM USART1 pins. The device life cycle and OTP settings determine which ROM boot paths are available.
5. For flash boot, remove both J4 shunts. The resistors select BOOT0=0 and BOOT1=0. Use a first-stage bootloader with the STM32 header and a board-specific external loader for the MX25UM51245G. The ROM reads serial NOR on XSPI1 through XSPIM port 2. Use the default single-SPI protocol for first bring-up. Verify the XSPI loader and timing before enabling octal or double data rate modes.
6. The board has a 48 MHz HSE crystal. Use the internal clocks for initial SWD checks and verify the crystal starts before relying on it. C7 and C8 are initial load values from the Nucleo circuit. Adjust after measuring crystal loading and startup margin. R10 is a DNP link for a clock source at TP12. Remove Y1 if you fit R10 and configure HSE bypass. Use LSI when the firmware needs a low-speed clock.
7. Configure PG8 as an output for the trigger on CW312 B19. Set CW313 JP2 to GPIO4 and configure the capture-side pin as an input.

## Mechanical and layout checks

The 64 contact pads match the template and RP2350 pad dimensions and center offsets to 0.0001 mm resolution. The 1.00 mm pitch increases to 3.00 mm across the key. The tongue span is 34.30 mm. The contact lands are 0.70 × 4.20 mm. The A row is on the back and the B row is on the component side. The card is 1.60 mm thick for the CW313 Samtec -02 slot. The board outline and contact positions follow the CW312 template. The nearest fitted component is more than 4 mm from the end of a contact land. Confirm seating, insertion force, and probe access with a first article.

The BGA has a 14 × 14 mm body, a 0.80 mm grid, 0.40 mm lands, and 0.47 mm mask openings. The layout uses six copper layers to escape the active BGA rows, provide ground references on layers 2 and 5, and distribute the auxiliary and core supplies on layers 3 and 4. The two inner ground layers also carry some routing. The reference planes must pass a return-path review.

KiCad 10.0.6 ERC and DRC report zero violations. DRC also reports zero unrouted pads and zero schematic parity issues.

The serial NOR has the same memory family and port assignment as the Nucleo design. This board uses the orderable SOIC-16W variant, with a 133 MHz part rating. The routed CLK, DQS, and IO0–IO7 copper lengths range from 6.10 to 25.26 mm, before via and package lengths. The Nucleo reference targets 1.27 mm matching for high-speed operation. The current routing requires length and return-path review before release. Confirm the PCB stackup and evaluate the flash timing at the intended clock rate with the selected assembly process. Review the core and auxiliary copper for IR drop at startup and full load. Verify the SMPS feedback route and snubber layout. Validate BGA solder mask and drill tolerances with the fabricator. No fabrication files are issued during the hold.

For the first article, verify the sequenced rise and discharge of all rails, HSE startup, SWD attachment, USART1 serial boot, flash identification and erase/write/read, flash boot, and current at idle and under load. Also verify the behavior when external signal cables are connected and the target powers down.

## References

- NewAE CW312 template rev 0, commit `40c337d2c8c15345bab03c9aa2072a23a36ccc3b`. The template license field says TBD. The CW312T_RP2350 rev 02 project is marked NDA. Contact and outline dimensions were measured and redrawn in KiCad.
- NewAE CW313 rev 02 schematic and PCB, commit `7b689277ca6852fccd195c29525a26640b6296d0`. The schematic title block states CC SA 3.0. [CW313 design files](https://github.com/newaetech/chipwhisperer-target-cw313/tree/7b689277ca6852fccd195c29525a26640b6296d0)
- Samtec PCIE-XXX-02-X-D-RA series drawing, sheets 1 and 2. [Connector drawing](https://suddendocs.samtec.com/prints/pcie-xxx-02-x-d-ra-mkt.pdf)
- ST MB1940-N657X0Q-C02, rev C-02. Sheets 3–5, 9, 12, and 14 cover I/O, power, flash, reset, and debug. The cover refers to ST's Open Platform License Agreement. The design uses reference circuit information and original KiCad artwork. [Nucleo schematic](https://www.st.com/resource/en/schematic_pack/mb1940-n657x0q-c02-schematic.pdf)
- ST DS14791 rev 11, plus the rev 2 package drawing and table 123 for the VFBGA264 land pattern. [STM32N657 datasheet](https://www.st.com/resource/en/datasheet/stm32n657x0.pdf)
- ST STM32 Open Pin Data, `mcu/STM32N657X0HxQ.xml`, commit `45c04599e2e0a788c5ed9b6218dc5a28a36041f7`. The symbol and footprint use the package's ball names and positions. See the included BSD 3-Clause license. [Pin data](https://github.com/STMicroelectronics/STM32_open_pin_data/blob/45c04599e2e0a788c5ed9b6218dc5a28a36041f7/mcu/STM32N657X0HxQ.xml)
- ST AN5967 rev 6, sections 3.4 through 3.6, for the power groups and timing. [Hardware development guide](https://www.st.com/resource/en/application_note/an5967-getting-started-with-hardware-development-for-stm32n6-mcus-stmicroelectronics.pdf)
- ST UM3234, serial NOR boot and USART1 pin assignment. [Boot ROM guide](https://www.st.com/resource/en/user_manual/um3234-how-to-proceed-with-boot-rom-on-stm32n6-mcus-stmicroelectronics.pdf)
- ST AN6000, core load transient specification. [Power supply guide](https://www.st.com/content/st_com/en/technical-documents/an6000-how-to-build-the-discrete-power-supply-for-stm32n6-mcus-stmicroelectronics.html)
- Macronix PM2183 rev 1.6, pin assignments, power, and ordering information. [Flash datasheet](https://www.macronix.com/Lists/Datasheet/Attachments/9241/MX25UM51245G%2C%201.8V%2C%20512Mb%2C%20v1.6.pdf)
- Texas Instruments SLVSD76C, TPS22918 load switch with QOD. [Load switch datasheet](https://www.ti.com/lit/ds/symlink/tps22918.pdf)

The ST-hosted PDF downloads for the Nucleo schematic and current datasheet, hardware guide, and boot ROM guide failed in this environment. Their contents were reviewed online. The local reference directory includes the available older ST revisions and the other downloaded reference PDFs.
