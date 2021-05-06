# Arduino JAMMA Game

Make a JAMMA game using only an Arduino and some leads.

## Disclaimer

Arduino, JAMMA, Atmel/Microchip, etc. do not officially endorse this library.  Use at your own risk.

## Project Documentation

The JAMMA connector is a double-sided PCB edge connector with 28 pins per side, introduced in 1985.  This standardizes the mechanical connection between an arcade cabinet and its installed game, allowing for some degree of upgradeability; in theory, new games could be installed into existing arcade cabinets, saving arcade operators the cost of a new cabinet when new games are released.  In practice, the specification is very loose electrically and there any many game/cabinet combinations which either do not function or can damage one another.  Consult this documentation to ensure that your cabinet is compatible before connecting the Arduino, but bear in mind that JAMMA compatibility is a complex matter.

The connector's pinout is as follows ([source](http://www.cosam.org/projects/supergun/jamma.html)):

|    | Solder Side                  | Component Side               |    |
| -- | ---------------------------- | ---------------------------- | -- |
| A  | Ground                       | Ground                       | 1  |
| B  | Ground                       | Ground                       | 2  |
| C  | +5V DC from cabinet to game  | +5V DC from cabinet to game  | 3  |
| D  | +5V DC from cabinet to game  | +5V DC from cabinet to game  | 4  |
| E  | -5V DC from cabinet to game  | -5V DC from cabinet to game  | 5  |
| F  | +12V DC from cabinet to game | +12V DC from cabinet to game | 6  |
| H  | (board orientation key)      | (board orientation key)      | 7  |
| J  | P2 Credit Counter            | P1 Credit Counter            | 8  |
| K  | P2 Credit Lockout            | P1 Credit Lockout            | 9  |
| L  | Speaker -                    | Speaker +                    | 10 |
| M  | (reserved)                   | (reserved)                   | 11 |
| N  | Video Green                  | Video Red                    | 12 |
| P  | Video Sync                   | Video Blue                   | 13 |
| R  | Service                      | Video Ground                 | 14 |
| S  | Tilt                         | Test                         | 15 |
| T  | P2 Credit                    | P1 Credit                    | 16 |
| U  | P2 Start                     | P1 Start                     | 17 |
| V  | P2 Up                        | P1 Up                        | 18 |
| W  | P2 Down                      | P1 Down                      | 19 |
| X  | P2 Left                      | P1 Left                      | 20 |
| Y  | P2 Right                     | P1 Right                     | 21 |
| Z  | P2 A                         | P1 A                         | 22 |
| AA | P2 B                         | P1 B                         | 23 |
| AB | P2 C                         | P1 C                         | 24 |
| AC | P2 D                         | P1 D                         | 25 |
| AD | P2 E                         | P1 E                         | 26 |
| AE | Ground                       | Ground                       | 27 |
| AF | Ground                       | Ground                       | 28 |

### Power

Many cabinets will not supply the -5V DC rail ([source](https://forums.arcade-museum.com/threads/the-mystery-of-5-volts.419936/)).  This library does not require it, but many classic games do.

Avoid connecting the JAMMA connector's +5V DC to the Arduino while simultaneously powering it through VIN, USB or 5V.  This should be supported, but Arduino clones do not always include the power management hardware needed to handle this safely.  Additionally, this may cause the cabinet to be driven through the Arduino, which is not designed to supply so much current.  The cabinet will also most likely not be designed to run on only +5V DC.

-5V DC or +12V DC will damage the Arduino and possibly a connected computer.  Ensure that these pins are not connected to the Arduino.

### Video

Video is unfortunately one of the electrically loose aspects of the specification.  The Video Sync pin controls the horizontal and vertical synchronization of the display, while the Video Red, Video Green and Video Blue pins control the brightnesses of the corresponding channels for the current pixel.

The precise voltages and signalling scheme are not part of the specification.  This board drives all video pins at +5V; ensure that your cabinet is configured to accept this.

Video timings can be adjusted, but, the following are recommended ([source](https://www.retrorgb.com/osscarcadetimings.html)):

Refresh rate: 60Hz

| Measurement     | Display Pixels | Description                                                                                                                                         |
| --------------- | -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Front Porch     | 12             | Blank space after active video and before horizontal sync.                                                                                          |
| Horizontal Sync | 31             | Video Sync is pulsed to mark the start of a line.                                                                                                   |
| Back Porch      | 62             | Blank space after horizontal sync and before active video.                                                                                          |
| Active Video    | 320            | Voltages on Video Red, Video Green and Video Blue control the brightnesses of their respective color channels as the beam scans from left to right. |

| Measurement   | Display Lines  | Description                                                                                                           |
| ------------- | -------------- | --------------------------------------------------------------------------------------------------------------------- |
| Front Porch   | 15             | Lines featuring only horizontal sync (timings unchanged) after active video and before vertical sync.                 |
| Vertical Sync | 3              | Lines featuring an inverted, reversed horizontal sync pulse (a long pulse with a short break at the end of the line). |
| Back Porch    | 18             | Lines featuring only horizontal sync (timings unchanged) after active video and before vertical sync.                 |
| Active Video  | 240            | Lines featuring horizontal sync and active video.                                                                     |

Note that due to the limited speed of the Arduino, a pixel produced by a game based upon this library will occupy multiple display pixels.

TODO: what is the factor?

The Video Sync pin must be connected to an otherwise unused GPIO pin with a 16-bit timer (such as D9 or D10 on an Arduino Uno).  The Video Red, Video Green and Video Blue pins can be connected to any otherwise unused GPIO pins which occupy the same port.  For the Arduino Uno, the ports are as follows:

| Port | Pins                           |
| ---- | ------------------------------ |
| B    | D8, D9, D10, D11, D12, D13     |
| C    | A0, A1, A2, A3, A5, A5         |
| D    | D0, D1, D2, D3, D4, D5, D6, D7 |

Note that connecting them to D0, D1 or D13 on an Arduino Undo may cause strange graphics during start-up or programming due to these being used by the Arduino boot-loader.

#### Implementation

TODO

### Audio

Audio is the only element which will require additional components to drive a cabinet.  This library can produce pulse width modulated audio on any otherwise unused GPIO pin with an 8-bit timer (such as D3, D5, D6 or D11 on an Arduino Uno).

If the cabinet does not include an amplifier, it should have an 8Ω speaker connected across Speaker + and Speaker -.  You will need a low-pass filter to remove the carrier wave and an appropriate amplifier between the Arduino's PWM output and the cabinet.

If the cabinet includes its own amplifier, only the low-pass filter and a DC blocking capacitor should be required.  Depending upon the amplifier's design, the wave may also need to be attenuated.

#### Implementation

TODO

### Input

The following JAMMA pins are considered inputs, and are initially floating, but are grounded by the cabinet when the associated button is pressed:

- Service
- Tilt
- Test
- P2 Credit
- P2 Start
- P2 Up
- P2 Down
- P2 Left
- P2 Right
- P2 A
- P2 B
- P2 C
- P2 D
- P2 E
- P1 Credit
- P1 Start
- P1 Up
- P1 Down
- P1 Left
- P1 Right
- P1 A
- P1 B
- P1 C
- P1 D
- P1 E

Any otherwise unused digital (D*) or analog (A*) GPIO Arduino pin can be assigned to any of these JAMMA pins, with some exceptions on an Arduino Uno; D1 and D13 used by the Arduino boot-loader during startup, which could risk a short from GPIO to ground, permanently damaging the Arduino and/or the cabinet.  D0 is similarly used, but through a current limiting resistor, which should not be a problem.

The following pins are considered to be an extension of the JAMMA standard, and may not be available on your cabinet:

- P2 D
- P2 E
- P1 D
- P1 E

#### Implementation

TODO

#### Miscellaneous Inputs

These inputs are part of the standard, but are unlikely to be needed.

##### Service

The service input usually grants a free credit without incrementing coin counters.

##### Tilt

This would be connected to a tilt switch which can set off an alarm should the cabinet detect risk of physical damage (such as tilting the machine).  Few games implement this.

##### Test

Generally, the test button triggers a reboot into a diagnostics screen.  This may be a latching switch rather than a momentary button in some machines.

### Events

#### Setup

This event is raised before the game starts.  Use it to initialize any game state.  Do not read from inputs here.

##### Example

TODO

#### Frame

This event is raised once per frame, shortly after the last line event for a frame.  There is time to update game state before the next active video line.  Note that the sample event may be handled, multiple times, during this event's handler.

##### Example

TODO

#### Line

This event is raised before each (game) line.  This should be used to prepare for it.  There is not much time available; taking too long may cause graphical glitches.  Do not read from inputs here.

##### Example

TODO

#### Sample

This event is raised at approximately 16560Hz, and should be used to output audio.  There is not much time available; taking too long may cause graphical glitches.  Do not read from inputs here.  Note that this may be executed during an interrupt partway through the frame event handler.

##### Example

TODO

### Unimplemented Functionality

These would only be useful in a real arcade, and likely require driver electronics.

#### Counters

P1 Credit Counter and P2 Credit Counter would normally be connected to mechanical counters inside the cabinet; when pulsed (unfortunately voltage and amperage are not specified) these would increment to track the number of credits inserted.

#### Lockouts

P1 Credit Lockout and P2 Credit Lockout would control solenoids which block the credit slot to prevent credits being inserted when the game is in particular states.  Few games appear to have used this, and voltages and amperages are not specified.
