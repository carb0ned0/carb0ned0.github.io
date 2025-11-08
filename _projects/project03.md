---
layout: page
title: Cognitive Radio for Spectrum Sensing
description: Developed a practical, low-cost testbed using SDR and GNU Radio to experimentally validate energy detection-based spectrum sensing for Cognitive Radio networks.
# img: assets/img/12.jpg
importance: 3
category: work
related_publications: true
---

## Overview/Problem Statement

The radio-frequency spectrum is a finite and valuable resource, yet much of it is inefficiently used. Traditional, fixed frequency licensing means many bands are "occupied" on paper but are often unused in reality, leading to artificial spectrum scarcity. The challenge is to create a system that can intelligently find and utilize these temporary "spectrum holes."

This project's goal was to design and build a physical, **low-cost testbed for Cognitive Radio (CR)**. The system needed to autonomously perform **spectrum sensing**—specifically, to detect which parts of the spectrum are busy (occupied by a "Primary User" or PU) and which are vacant. As a primary researcher on this paper , my role was to design the system architecture, implement the signal processing flow-graphs in GNU Radio , conduct the physical experiment using SDR hardware , and analyze the detection results.[^Paper]

[^Paper]: {% cite 10.1007/978-981-99-2710-4_24 %}

## Process/Research

Our methodology was based on **Energy Detection (ED)**, a common and effective spectrum sensing technique. The ED process is straightforward: capture a signal from a specific frequency band, measure its received energy, and compare it to a predetermined threshold.

* If **Energy > Threshold**, the band is considered "Occupied" by a Primary User.
* If **Energy < Threshold**, the band is considered "Vacant" and available for a "Secondary User" (SU).

We designed a two-part experimental system:

1. **The PU Transmitter**: A simulated "Primary User" to represent a licensed broadcaster. We modeled this using four separate signal sources at distinct frequencies (20KHz, 45KHz, 70KHz, and 100KHz) to create a known "ground truth" of occupied channels.
2. **The SU Receiver**: The "Secondary User" or "Cognitive Radio". Its entire purpose was to scan the environment and correctly identify the frequencies being used by the PU transmitter.

## Solution/Implementation

The core of the solution was implemented using two **GNU Radio flow-graphs**, which visually represent the signal processing chain.

### 1. PU (Transmitter) Flow-graph

* We used four `Signal Source` blocks to generate simple cosine waves at our target frequencies.
* These signals were combined using `Add` blocks.
* The final combined signal was fed into an `Amitec Sink` block, which acts as the driver for the SDR hardware, transmitting the signals over the air.

### 2. SU (Receiver) Flow-graph: This was the "sensing" part of the system

* An `Amitec Source` (the receiving SDR) captured the live RF environment.
* The data was converted from a stream to vectors (`Stream to Vec`)  and fed into an `FFT` (Fast Fourier Transform) block. This transformed the raw, time-domain signal into the frequency domain, allowing us to see the power at each specific frequency.
* A `Complex to Mag^2` block calculated the power spectrum (the "energy") of the signal.
* This power data was then compared against a threshold (the ED logic) , and the results—the frequencies of the detected "busy" channels—were written to a `File Sink` for analysis.

This entire system was physically implemented using two Amitec SDRs and two PCs, one for transmitting and one for receiving.

## Results/Impact

The testbed was a success. Our system was able to **accurately detect the occupied PU channels "almost perfectly" and with an "insignificant delay"**.

The output from the File Sink clearly showed detected energy in the exact frequency ranges we were transmitting. For example, our 45KHz signal was correctly identified in the 45396.0 Hz to 45864.0 Hz range.

However, the results also highlighted a key challenge: we observed some **false alarms**, where the system incorrectly flagged an empty channel as "busy". This is a classic trade-off in sensing. This finding emphasized that the **choice of threshold is the most critical factor** for performance. A threshold set too low causes false alarms (wasting spectrum), while one set too high causes misdetections (causing interference).

As a future improvement, I would explore dynamic threshold algorithms that adapt to background noise, rather than a fixed value, and implement noise cancellation mechanisms.

## Tech/Reflection

* Technology Stack:
  * Hardware: Amitec Software Defined Radios (SDRs).
  * Software: GNU Radio  (an open-source toolkit for signal processing).
  * Language: Python  (used to create custom GNU Radio blocks and for final data analysis).
* Why this stack?
  * SDR was essential because its behavior is defined by software, not fixed hardware. This flexibility is the entire basis for Cognitive Radio.
  * GNU Radio is the industry standard for this. Its visual, flow-graph-based approach allowed us to rapidly prototype, test, and modify complex signal processing chains without writing low-level C++ for every component.

This project was a fantastic bridge between abstract communication theory and practical hardware implementation. The biggest challenge was managing the imperfections of real-world radio signals, which is something a simulation can't teach you. The final result was a proven, functional, and low-cost testbed that can be used as a platform for future Cognitive Radio research.

> Code and design files is yet to be uploaded.
