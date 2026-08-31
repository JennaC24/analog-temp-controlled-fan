# analog-temp-controlled-fan
Analog temperature-controlled fan system designed with a TMP36 temperature sensor, differential and non-inverting amplifiers, second-order active filtering, a Wien bridge oscillator, and transistor-based fan control.

## Table of Contents
[The Problem](#the-problem)  
[What I Built](#what-i-built)  
[System Design](#system-design)  
[Circuit Design](#circuit-design)  
[Components & Tools](#components-and-tools)  
[Simulation & Validation](#simulation-and-validation)  
[Demo](#demo)  
[Results](#results)  
[Challenges & What I Learned](#challenges-and-what-i-learned)  
[Repository Contents](#repository-contents)  
[References](#references)  
[About Me](#about-me)

## The Problem
Electronic systems can require active cooling when operating temperatures increase. A continuously running fan provides cooling regardless of the current temperature, while manual control requires someone to monitor the temperature and adjust the cooling system. The goal of this project was to develop a system that could automatically respond to changes in temperature and activate a cooling fan when additional cooling was needed. 

Rather than relying on a microcontroller, the project was implemented using analog signal processing and discrete components, requiring the temperature signal to be sensed, conditioned, filtered, amplified, and converted into a signal capable of driving a 5 V fan.

## What I Built
I designed and implemented an analog temperature-controlled fan system that uses a TMP36 temperature sensor to measure temperature and a series of analog circuit stages to process the sensor output and control a 5 V brushless DC fan.

The final system uses a differential amplifier, second-order active low-pass filter, non-inverting amplifier, Wien bridge oscillator, comparator, and transistor driver to condition the temperature signal and provide the current and switching signal required by the fan.

The project was developed iteratively through three milestones, beginning with a simple temperature-triggered LED and progressing to the final fan-control system.

[add block diagram here]

## System Design


## Circuit Design

## Components and Tools

## Simulation and Validation

## Demo

## Results

## Challenges and What I Learned

## Repository Contents

## References

## About Me
**LinkedIn:** [linkedin.com/in/jenna-connelly](https://www.linkedin.com/in/jenna-connelly-42a4a73a4)\
**Email:** [jconnel24@gmail.com](mailto:jconnel24@gmail.com)
