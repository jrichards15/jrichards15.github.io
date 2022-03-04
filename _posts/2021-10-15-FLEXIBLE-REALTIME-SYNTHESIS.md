---
title: STM32 LCD User Interface
date: 2021-10-15 08:00:00
layout: page
published: true
---

## Flexible, Realtime Audio Synthesis
As I've mentioned before, I was working on another project in the background while getting the LCD and encoder working for a UI. As a fan of modular synthesis, I wanted to create a highly-configurable scheme for patching different sound sources and audio effects into a cohesive chain. To this effect, I integrated an I2S-compatible audio-quality DAC for outputs. It's stereo-compatible, providing 2 channels for audio, which means I can support two simultaneous audio streams (left and right). The DAC is driven by a DMA stream of audio samples, which means its drivers require very little processing time.

In this blog post, I will use "sound source" and "voice" interchangeably. In addition, I also designed a bunch of audio effects that can be chained to add a variety of possible sounds to the device without a lot of additional code. 

### General Voice Design
Starting out, I wanted to use the object-oriented features of C++ to allow for future expansion and compatibility, which was easily achieved by creating a virtual base class from which all of the different sound source types could be defined. To this effect, each voice type must have a number of functions and members. Each voice contains a function that generates audio samples as they would appear in time, meaning each one will produce consistent output synchronized to the frequency of the output audio stream. There are additional functions for initializing the class's members and adjusting the unique parameters for each model, though these will be subclass-specific and defined on a per-voice bases. In addition, each subclass has its own UI tables (discussed in a previous blog post) that must be accessible to the UI handler. As of this moment, there are a number of voices that I have finished implementing:

* Karplus-Strong: A commonly-implemented synthesis algorithm for generating realistic string sounds with low overhead. It is well-documented and versatile, and I have implemented a slightly more advanced version that allows for timbre control, frequency adjustments, and variable decay times. At its core, Karplus-Strong uses a ring buffer filled with random noise, which is then fed through an averaging filter to decay the noise slowly, generating a pitch that is directly related to the length of the delay line (or ring buffer). Each of these properties has been parameterized and broken out to the LCD user interface, allowing the user fine control over the properties of the algorithm. These controls can vary the output signal's overall sound, allowing for the generation of realistic guitar sounds and harsher, distorted sounds.
* Wavetable: Another common algorithm! This one utilizes a table of pre-computed values for a single period (or longer) for the desired waveform. The values of the table can be interpolated between, allowing the frequency to be varied by changing the increment rate of the index for the table. This allows for a huge variety of complex sounds with very little computational overhead, making it perfect for emulating analog (or simply more complex) synthesis methods.
* LFSR Noise: This one provides a very fast method of generating white noise using only integer operations and no complex (or hardware) random number generation techniques. This method is commonly used in simple circuits, creating long strings of pseudorandom numbers with almost no overhead. The output sounds nice enough for our purposes, and executes very quickly. Coupled with some other effects and maybe an envelope generator, this can be turned into a very satisfying snare drum sound.
* FX Chain: This one is the most complicated, but it allows for infinite-length chains of sound sources and audio effects. The overall function and structure of this voice is elaborated upon in a later section of this post.

### General FX Processor Design
This is fundamentally similar to the voice design, utilizing the same inheritance relationship as previously discussed. Each effect requires an input source, a method to set the input source, and a function to generate sequential samples of the processed signal. On top of these, each subclass can define its own parameters, setter functions, and UI-related definitions. The UI table allows for the UI handler to enable user control of a derived effect's parameters. As of right now, here are the effects I have implemented:

* Envelope Generator: This effect was a quick and simple way of integrating a system for playing fixed-length notes by varying the volume of the output. The envelope implemented here provides a linear ADSR, where the volume levels for 4 different stages can be defined by a linear relationship. It allows each stage to have a variable length and volume slope. The attack stage is linearly interpolated to the highest volume, then decreasing the volume down to the defined decay volume. This is held for a variable duration ("sustain"), which then decays down to zero for the release stage. This creates versatile volume and duration options, enabling the creation of piano and drum-like sounds from the same code.
* Mu-Law Companding: This is a very cool effect, in my opinion. In attempting to emulate the sound of old cassette tapes, the algorithm here converts signed-16 bit samples down to a simple 8-bit signed output while preserving a substantial portion of the frequency spectrum. It introduces a lo-fi quality to the audio, and also requires very few computational resources as a bonus.
* Downsampler/Decimator: This one holds a single input sample for a fixed duration, allowing for the harmonic distortion caused by lowering the Nyquist frequency. It's a very neat little effect, and it saves processing time by requiring the computation of fewer samples. 
* Bitcrusher: This is a simple method for giving the audio a lower-quality sound. It lowers the bit resolution of the samples, only preserving the N most significant bits of every sample, along with the sign bit. At the lowest bit depth, the audio is still comprehensible, though there is a high level of distortion that makes the audio slightly unpleasant. For more moderate settings, the output is relatively pleasing and achieves the desired aesthetic.

A lot of the defined effects allow for less computation or have smaller underlying hacks to reduce the computation required for samples. Since the clock speed of my STM32 is maxed out, the best way to increase the capabilities of the audio generation is to write algorithms that execute very quickly and minimize the number of CPU cycles required for computing each sample. The DAC is configured to expect 44.1kHz (CD-quality) audio, and generating samples at that speed requires the vast majority of overall processing time.

### Realtime FX Chain Structure
Now, there needs to be some way to configure these signal chains. The FX chain is implemented as a sort of quasi-linked list, where an input voice and output effect can be defined pairwise. Since all voices and effects are derived from the same base class, this uses polymorphism to its benefit, making it very flexible. In the same token, the FX chain voice is derived from the same class, meaning an infinite number of effects can be paired with a single input sound source, feeding all the way to the output. This method limits a lot of the complexity of configuring signal chains, and utilizes a UI scheme for configuring all of the voices in a streamlined fashion. 

### Voice Management
Now, given the definitions for voices and effects, there needs to be a central controller for all of this information. The Voice Manager supports two signal chains using any of the voices, including the effects chain voice. In addition, it manages the speed of the output signal and provides it to the drivers for the DAC. It manages its own UI tables, allowing the user to navigate through the various screens with ease and configure their own signal chains without a lot of needless complexity.