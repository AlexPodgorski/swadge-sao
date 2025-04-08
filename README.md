# CR123A SAO

This is an evaluation board to test the viability of using LiMnO2 CR123A cells in a 1P3S configuration on the Swadge.

Attendees are reporting that their Swadges are too heavy. This is largely due to the 3xAA batteries, 24g each, used to power the Swadge in a 1P3S configuration. Batteries were evaluated, and CR123A batteries were selected for further evaluation on the SAO. They weigh in at 17g each.

CR123A are non-rechargeable, Lithium Chemistry batteries commonly used in camera and firearm accessory applications. They have a higher C rating than other traditional batteries (thanks to the Lithium Chemistry) and are safer than a traditional Lithium Ion or Lithium Polymer rechargeable battery. They maintain a steady voltage level of ~2.9V and discharge curves suggest approximately a 12hr battery life at 100mA continuous draw.

Preliminary testing showed:
1. Big Bug was pulling ~225mA
2. Swadge hero ~90% brightness with light show ~100mA
3. Main menu ~90mA-100mA

This is with the MT3410 + 5V USB input. 225mA at 5V is a whopping 1.125W. If we assume that same power draw and change to a 1P3S 9.0V source, we should expect to see 125mA draw (1.125W/9.0V) from the 3 cells.

The MT3410 does not support inputs above 6V, so a new DC-DC converter needed to be specified. XI’AN Aerosemi Tech has other PMICs for higher voltages; the most affordable one appropriate for our use case is the <a href="https://www.lcsc.com/datasheet/lcsc_datasheet_2408140941_XI-AN-Aerosemi-Tech-MT2492_C89358.pdf">MT2492</a>. 

The MT2492 operates at a different switching frequency than the MT2410. The MT2410 switches at 1.2MHz vs 600kHz at the MT2492. I have EMI concerns with this. Additionally, the manufacturer recommends upsizing the inductor under light load conditions (<100mA), which can also impact EMI. I don’t know if this will be “worse”, but it will be different. Predicting its behavior will be hard, so I’m just going to send it on this SAO and see what happens. I have broken out the inductor pads such that I can easily drop in a new one if I’d like to evaluate different values or if I need to probe something.

A lower switching frequency may be more efficient; I reviewed some <a href="https://www.ti.com/lit/an/slvaed3a/slvaed3a.pdf?ts=1744087588249&ref_url=https%253A%252F%252Fwww.google.com%252F"> documentation from TI </a> discussing how switching frequency impacts the performance of a buck converter. TI offers a device with adjustable switching frequency between 600kHz and 1MHz, and showed that at all loads, the 600kHz configuration ran cooler and saw ~85% efficiency vs 70% at loads we typically see in the Swadge. This is fairly device specific, but comparison of the Aerosemi datasheets suggests a higher efficiency as well with the 600kHz chip. Frankly, the plots on the datasheets are horrible and make drawing conclusions difficult. The MT2492 datasheet suggests that efficiency of the IC is around 95% at 9V/1A, and efficiency drops to around 93% at lower loads (~200mA). The MT2410 peaks at around 93% at 5V/300mA, and floats around 85% at 200mA. So, I am expecting about an 8-10% increase in efficiency switching to the MT2492.

The drawback in switching to a 600kHz solution is the requirement to upsize capacitors and inductors, which increases PCB area required for the MT2492. Higher frequency switching converters perform better in transient testing, so the MT2492 may over/undershoot during transition states more than the MT3410 does; possibly 15mV more? This is why larger capacitors and inductors are required.

A shunt Diode is in parallel with each battery into the MT2492 circuit. This is the only “battery management system” on board. It is possible (but unwise) to put in the RCR123A batteries (rechargeable Lithium Ion) although they have the same form factor, as there is no individual cell monitoring on this board.

# Testing Plan

The testing plan for this SAO is in two phases.

First is to attach it to a Hot Dog and monitor current draw from the USB on the SAO. This is to accomplish two tasks:
1. Compare against the existing Swadge circuit to evaluate power efficiency and noise of this new voltage regulation circuit
2. Gather more benchmark data to evaluate the battery performance

The states to be evaluated are:

1. Idle on Main Menu at max LED and TFT brightness, no screensaver
2. Big Bug at max brightness, maximum volume, general gameplay for 30 minutes and logging current consumption approximately every 60 seconds
3. Colorchord at 7 gain, LED 8, Max Screen Brightness Rainbow
4. Stepwise 0-7 TFT brightness on Main Menu
5. Stepwise 0-8 LED brightness on Main Menu
6. UTTT Wireless Connect Menu at max LED and TFT brightness


In theory, this will help us characterize current draw in various Swadge modes and help estimate run time in hours.

Second is to install the batteries and:
1. Monitor and log voltage at the diode node to GND over time
2. Monitor and log cell voltage at each node to GND over time

This will discharge a few sets of CR123A batteries in a few different states to characterize their charge time and see if the cells are staying relatively balanced. The states evaluated will be:
1. Big Bug at max brightness, maximum volume, general gameplay
2. Idle on Main Menu at max LED and TFT brightness, no screensaver
3. UTTT Wireless Connect Menu at max LED and TFT brightness

Durations for these tasks will be chosen based on the results of the first set of tests.

