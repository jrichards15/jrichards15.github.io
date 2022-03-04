---
title: STM32 LCD Experiments
date: 2021-09-16 08:00:00
layout: page
published: true
---

## Simple Drivers to Control an HD47780-Compatible Display
As an embedded developer, I wanted some additional practice working with the many peripherals present on the STM32H7 line of products. Using an STM32H745ZI Nucleo board, I prototyped a number of projects on a breadboard using open-source hardware from Adafruit. While not the most low-level and design-centric approach, I felt that using well-tested breakouts was the fastest approach to achieving the desired goals of this project.

### Hardware
The hardware configuration for this project was fairly straightforward, as I utilized the I2C communication protocol to conserve pins, as this blog post pertains to a small portion of a much larger project. The I2C peripheral on the STM32 is used to interface with an 8-bit GPIO expander IC, the MCP23008. It's quite a straightforward IC with a thorough datasheet and many well-utilized projects. All of the LCD pins (excluding power and ground) were tied to a different GPIO output pin on the expander to streamline the hardware drivers.

The GPIO expander was only the first portion of this project, as the LCD drivers were slightly more complex. The HD44780-type character LCD I used was 2-line, 16-character display. The interface itself is fairly simple, using a level-sensitive enable pin in 4-bit mode in order to properly control the display. Since the display pins were on the GPIO expander, I was able to use simple I2C controls, rather than individually setting and unsetting the individual pins manually on the STM32.  

### Software
For my purposes, I cross-referenced the Arduino driver code provided by Adafruit and converted it into low-level C++ compatible with my Nucleo board. By emulating other functional drivers, I was able to effectively cut my development time down significantly. These were fairly simple to implement using the manufacturer-provided HAL drivers for my selected chip. 

The I2C peripheral is configured in polling (blocking) mode, which results in some slowdown when performing normal operations on the chip, but overall works fine and implicitly fufills some of the timing requirements for the LCD. For a simple experiment, this is sufficient for my purposes. When expanding this into a larger project, I may switch to utilizing interrupt-driven or DMA-based I2C, though I was not intially getting good results with either of these methods. 

By extension, the LCD code wraps up the MCP23008 drivers into high-level code that allows for commands to be sent and characters to be written without directly interfacing with the hardware itself. This is meant to simplify the higher-level and application code, which it achieves fairly well. Both the HD44780 and MCP23008 classes utilize the singleton design pattern to avoid potential issues when attempting to use the same hardware resources on multiple instances of their classes. This simplifies a lot of the control code, and produces well-integrated results into other projects.

### Closing Thoughts
This project helped me with my design process for low-level drivers, and allowed me to consider what I appreciate in the open-source code that I adapt for other projects. As such, I have been considering the best way to package my STM32CubeIDE project for public consumption, and for now I plan to clean up the code so it's easier to read and understand. As such, I will not be making a publically-available repository until this work is finished. 

It's remarkable to see a bunch of unrelated hardware jump into action after writing a few hundred lines of code, and utilizing the LCD allows for some truly fantastic results with a simple interface. Since beginning this project, I have found a larger 4-line by 16-character display with an RGB backlight, which would provide a flashier, more easily-readable interface. I want to eventually integrate this with my existing drivers, though it is not my highest priority at this moment. This was very satisfying to work on, and will provide a scaffold for future projects that allow a user to directly control the actions of the chip. For more development on this project, please check out my more recent blog posts to see how this progresses over the coming months. 

