# Notes for second synth[*] layer

This patch adds six more synth slots and six more pattern slots.

[*] and pattern

## Installing extra synths and patterns

To install the extra synths add more synth modules with folder names
`7-...` to `12-...`. Note that you need a hyphen immediately after the slot
number.

Installing extra patterns is exactly the same, but in the patterns folder.
Again, make sure you use hyphens in folder names.

## Use

To access one of these synths hold down the shift button and double tap one
of the six synth buttons. For example, if you hold shift and double tap
synth button 1 (B below middle C) you get synth 7.
If you hold shift and double tap
synth button 2 (middle C) you get synth 8. And so on.

Similarly, use a double tap to access patterns 7 to 12.

You don't need to fill all twelve pattern or synth slots. But if you
try to use a slot that isn't filled then you won't get any sound.

The double tap time is 500ms. That's the time required from pressing
a button down the first time, to pressing the same button down a second time.

## Blinking LEDS

If you select a pattern or synth from the second layer then its
corresponding LED will blink.

If you don't like the blinking you can turn this off by editing the
mother patch.
For pattern blinking go [pd leds] > [pd pattern-leds] >
[pd blink-second layer] and follow the comments.
For synth blinking go [pd leds] > [pd synth-leds] >
[pd blink-second layer] and follow the comments.

## Changes

The changes to allow the second synth layer are all in mother.pd:
* In [pd synth-select] there is a new patch [pd synth-select-layer].
* In [pd synth-select] there is a new selection object to load synths 7 to 12.
* In [pd auto-load-synths] we change [i 6] to [i 12] to load more synth modules.
* In [pd auto-load-synths] we send more loadbangs (they go to the new synth modules).
* In [pd auto-load-synths] the [makefilename] object loads filenames that start with `%d-*` instead
of just `%d*`, thatk a folder called `10-MySynth` doesn't get confused with
`1-MyOtherSynth`.
* In [pd leds] > [pd synth-leds] there is a new patch [pd blink-second layer].

The changes to allow the second pattern layer are exactly the same as for the
second synth layer, but for the pattern patches.

## Important notes

This is a direct fork of v1.2 of the 201 Pocket Piano software. That means
that either you should install all the files, or---if you just want to
copy over mother.pd---you should be copying it over the default v1.2
software. Otherwise it may not work as expected, and if you've made your own
changes to v1.2 you will lose those.

Beware that loading six more synths will add to the device's startup time.
This may or may not be noticeable. It also depends on which synths
you choose.


# 201 PocketPiano

The software for the pocket piano consists of two parts: a PureData (Pd) patch and a controller program. 

The Pd patch handles all the heavy lifting: audio synthesis, MIDI, sequencing, loading pattern and synth modules, etc. It is designed to be platform-independent; it can be run on any machine with Pd installed. However, it relies on several external objects that must also be installed.

The controller program provides the controls for the Pd patch: keys, knobs, and LEDs. The controller program and Pd communicate via Open Sound Control (OSC) messages. Currently, the controller program is a C program that runs on the hardware, in this case, an i.MX6 processor running Yocto Linux. It checks for key presses and knob movement and sends these events to the Pd patch as OSC messages. It receives OSC messages from the Pd patch to set the LEDs.

## Pd Patch 

On the Pd side, things are divided into two parts: a mother patch and pattern / synth module patches. The mother patch (mother.pd) communicates with the controller (via OSC) and handles MIDI, Presets, sequencing, and loading of the pattern and synth module patches. Extra files and helper patches are included in the lib folder.

The modules are designed to be self-contained patches that are dynamically loaded by mother.pd during initialization. To the mother patch, the module patches are black boxes: mother.pd sends MIDI notes and knob values in and gets sound out. For the modules, we decided to use the same format as Orac (a virtual modular synthesizer for the Organelle an other platforms). Orac can load modules for the 201 Pocket Piano, and modules may developed using the same method as one uses to create Orac modules. It should be noted that while modules for the 201 may be run inside Orac, it is not possible to run any random Orac module on the 201 without modification. The mother patch requires each module to have the exact same set of parameters, where in Orac, a module can have any number of parameters.   

## Creating Modules 

There are two types of modules that get loaded: patterns and synthesizers. mother.pd will load 6 pattern modules and 6 synthesizer modules during initialization. The module folders should be prefixed with a number 1-6 indicating the slot they will be placed in. Each module requires at least 2 files: module.pd and module.json. The module.pd file is the main entry point and is the file that gets loaded by mother.pd. The mother patch does not strictly require the module.json file, but it is required to be compatible with Orac. The module folder may also contain other sub-patches, sound files, or other data required by the module.  

The pattern modules are the most simple as they take no parameters. They recieve MIDI notes in [r notesIn-$1], apply some pattern process, and send MIDI notes out [s notesOut-$1]. The $1 part of the name becomes the slot number when the module is loaded and, therefore, unique for each module. Pattern modules also receive 2 global messages from the mother.pd patch: beat-clock and global-bpm. The mother.pd patch handles the metronome, and it sends a 0-360 integer beat phasor (beat-clock) and the current BPM (global-bpm). A module can use these to create arpeggios or other patterns based on the global clock.  

Synthesizer modules are only a little more complex. They receive MIDI notes [r notesIn-$1] and three knob parameters: [r knob1-$1], [r knob2-$1], and [r knob3-$1]. They will perform some synthesis or sampling and then send audio out: [outlet~ outL-$1]. 

The easiest way to start creating modules is by copying existing ones. The 1-thru module is a good starting point for pattern modules, and 1-RedMode is a good one for synth modules. 

As modules are loaded dynamically, it is important to avoid using the [loadbang] object for internal initialization. Instead, use [r loadbang-$1]. The mother.pd patch sends this bang after the modules are loaded, so using [r loadbang-$1] instead of [loadbang] will ensure everything gets initialized in the correct order. 

