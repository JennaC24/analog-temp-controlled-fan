# analog-temp-controlled-fan
Analog temperature-controlled fan system designed with a TMP36 temperature sensor, differential and non-inverting amplifiers, second-order active filtering, a Wien bridge oscillator, and transistor-based fan control.

## Table of Contents
[The Problem](#the-problem)  
[What I Built](#what-i-built)  
[System Design](#system-design)  
[Circuit Design](#circuit-design)  
[Components & Tools](#components-and-tools)  
[Simulation & Validation](#simulation-and-validation)  
[Design Evolution](#design-evolution)  
[Demo](#demo)  
[Results](#results)  
[Challenges & What I Learned](#challenges-and-what-i-learned)  
[Repository Contents](#repository-contents)  
[References](#references)  
[About Me](#about-me)

## The Problem
Electronic systems can require active cooling as operating temperatures increase. A continuously running fan provides cooling regardless of the current temperature, while manual control requires someone to monitor the temperature and adjust the cooling system. 

The goal of this project was to develop a system that could automatically respond to changes in temperature and activate a cooling fan when additional cooling was needed. Rather than relying on a microcontroller, the system was designed using analog signal processing and discrete components to sense, condition, filter, and amplify the temperature signal before using it to control a 5 V cooling fan.

## What I Built
I designed and implemented an analog temperature-controlled fan system that uses a TMP36 temperature sensor to measure temperature and a series of analog circuit stages to process the sensor output and control a 5 V brushless DC fan.

The final system uses a differential amplifier, second-order active low-pass filter, non-inverting amplifier, Wien bridge oscillator, comparator, and transistor driver to condition the temperature signal and provide the current and switching signal required by the fan.

The project was developed iteratively through three milestones, beginning with a simple temperature-triggered LED and progressing to the final fan-control system.

[add block diagram here]

## System Design
The signal flows through six stages, each conditioning the output of the last so it falls within the input range the next stage expects:

Sensing -> Comparing -> Filtering -> Amplifying -> Pulse-driving -> Output

The TMP36 produces a voltage that scales linearly with temperature. An OP484 differential amplifier subtracts a fixed reference voltage from that signal, so the output tracks how far above a baseline the temperature is, rather than switching between two fixed states like a simple comparator would. That signal is cleaned up by a second-order active low-pass filter, then boosted by a non-inverting amplifier to a range the fan stage can use. Because the fan is a DC motor with physical inertia and coil inductance, it can't be driven by a steady low-current analog signal without stalling. As a result, a Wien bridge oscillator and comparator convert the amplified signal into a high-frequency switching (pulsed) drive signal, and a transistor supplies the current the fan actually needs. The full derivation, values, and reasoning for each stage are below in [Circuit Design](#circuit-design).

## Circuit Design

The final (Milestone 3) circuit is built from the following stages. Values shown are the ones used in the final design unless noted otherwise.

### Temperature Sensor — TMP36

**Role:** Converts ambient temperature into a proportional voltage.

**Equation:** Vout = 0.01·T + 0.5 V, where T is temperature in °C (10 mV/°C scale factor, 0.5V offset)

**Values:** Supplied at 5V (Vs+), GND on leg 3, Vout on leg 2

**Why this component:** The TMP36 was chosen because its output scales linearly with temperature, which makes it straightforward to interface with downstream op-amp stages. Its −40°C to +125°C range comfortably covers room-temperature and elevated operating conditions, its built-in 0.5V offset keeps the output positive even at negative temperatures, and its accuracy (~±1°C) and 10 mV/°C scale factor made it reliable without needing extra calibration circuitry.

### Differential Amplifier — OP484

**Role:** Outputs the difference between the sensor voltage and a fixed reference, so the response scales continuously with temperature instead of snapping between two states like the comparator used in Milestone 1.

**Equation:** Vout = (R8/R7)(Vin+ − Vin−), with R7 = R8 so gain = 1

**Values:** Four 2.2kΩ resistors (equal, for unity gain); reference (Vin−) = 0.33V; V+ supply = 5V; V− = GND

**Why this component:**
The OP484 was used here (replacing the OP97 comparator from Milestone 1) specifically because it's rail-to-rail. The differential amplifier's output needs to represent low voltages accurately near 0V and scale cleanly up to 5V. A non-rail-to-rail op-amp like the OP97 can't reach or resolve those extremes correctly. Gain was deliberately kept at 1 at this stage; amplifying here would also amplify the noise the op-amp itself introduces, which is why amplification was pushed to a later, dedicated stage.

### Second-Order Active Low-Pass Filter — OP484

**Role:** Removes high-frequency noise introduced by the differential amplifier before the signal is amplified further.

**Equation:** Fc = 1/(2π√(R1·R2·C1·C2)), with R1 = R2 and C1 = C2

**Values:** R1 = R2 = 1kΩ, C1 = C2 = 4.7µF → Fc ≈ 33.9–34 Hz (calculated 33.9 Hz, simulated 33.95 Hz, measured 34 Hz)

**Why this component:** This replaced the first-order passive RC filter used in Milestone 2. The passive filter worked fine in isolation, but the stage after it would draw a small amount of current from it, shifting the effective cutoff frequency. An active filter isolates the RC network from that loading effect. Resistors and capacitors were kept equal specifically to hold the filter's gain at exactly 1. Gain above 1 makes a second-order active filter unstable, and above 3 it turns into an oscillator outright, so unity gain was a hard constraint here. The OP484 was used again for its rail-to-rail range, since the signal at this point is still well under 1V.

### Non-Inverting Amplifier — OP484

**Role:** Scales the filtered signal up to the 3–5V range the fan stage needs.

**Equation:** Vout = (1 + R4/R3)·Vin+

**Values:** R3 = 10kΩ, R4 = 47kΩ → gain = 1 + 47/10 = 5.7x

**Why this component:** An earlier version of this stage (Milestone 2) used the OP97 with a gain of 11x (R3 = 1kΩ, R4 = 10kΩ), which worked when the signal levels were in the 0–0.5V range feeding a 0–5V LED-driving output. In Milestone 3, the pre-amplified signal is under 1V and the OP97's non-rail-to-rail limitation produced incorrect output at these lower voltages, so the amplifier was rebuilt using the OP484.

### Wien Bridge Oscillator — OP97

**Role:** Generates a continuous high-frequency sine wave used to build a pulsed drive signal for the fan, since a DC motor's coil inductance and physical inertia cause it to stall under a low, steady drive voltage.

**Equations:** f = 1/(2π·R·C); Gain = 1 + R1/R4 (must exceed 3 for sustained oscillation)

**Values:** R2 = R3 = 6.8kΩ, C1 = C4 = 1nF → f ≈ 23.4 kHz (calculated and simulated); R4 = 6.8kΩ, R1 = 20kΩ → gain ≈ 3.95

**Why these values:** The oscillation frequency needed to land in the 20–25 kHz range so the fan's motor could "average out" the pulses into what feels like smooth, continuous drive. The gain resistors were chosen to sit just above the theoretical minimum of 3 required to sustain oscillation since below 3 the oscillation converges to 0, and choosing a value just above 3 lets the amplitude grow until it's capped by the supply rails rather than growing unbounded. Experimentally, the physical circuit only reached ~11.6 kHz instead of the designed 23.4 kHz, which was traced to the OP97's slow slew rate (0.2 V/µs) (it can't switch fast enough at that amplitude). This didn't end up mattering for the design, since the oscillator only needs to produce a clean, symmetric sine wave centered at 0V for the comparator stage to work.

### Comparator

**Role:** Converts the oscillator's sine wave into a square wave that switches the fan drive on and off at high frequency.

**How it works:** The oscillator's sine wave (centered at 0V) is fed into the comparator's inverting input, with the non-inverting input tied to ground. This makes the output switch between the amplifier's output voltage (V+) and 0V (V−, ground), producing a square wave whose "high" level equals the current temperature-scaled drive voltage.

### Transistor — TIP31C (NPN)

**Role:** Boosts current from what the op-amp stages can supply up to what the fan actually requires.

**Equation:** Iout = hFE · Ib, with hFE ≈ 25–50 and Ib ≈ 20–40 mA

**Why this component:** Before the transistor stage, the circuit could only supply 20–40 mA (enough for an LED, such as in Milestones 1 and 2, but far short of the ~200 mA the 5V DC fan needs to run). The TIP31C was chosen to bridge that gap, amplifying the available current past the fan's operating threshold without needing a redesign of the earlier signal-conditioning stages.

### Output — 5V DC Brushless Fan

**Role:** Provides physical cooling, turning on past the trigger point and scaling its effective drive with temperature above that.

**Specs:** Operates in the 3–5V range, draws ~200 mA, ~30mm × 30mm

Why this component: The fan needed to match the circuit's available voltage range (3–5V) without exceeding what the power supply could provide. A brushless design was chosen specifically because it has a longer operational life (no brushes to wear out) and produces less electrical noise than a brushed motor — which matters here since the TMP36's analog output is small and noise-sensitive. It's also simple to wire into a breadboard prototype, needing only power and ground.

The full LTspice schematic is linked in the Project Manual and included in this repository (see [Repository Contents](#repository-contents)).

## Components and Tools

## Simulation and Validation

## Design Evolution

## Demo

## Results

## Challenges and What I Learned

## Repository Contents

## References

## About Me
**LinkedIn:** [linkedin.com/in/jenna-connelly](https://www.linkedin.com/in/jenna-connelly-42a4a73a4)\
**Email:** [jconnel24@gmail.com](mailto:jconnel24@gmail.com)
