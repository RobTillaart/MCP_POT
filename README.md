
[![Arduino CI](https://github.com/RobTillaart/MCP_POT/workflows/Arduino%20CI/badge.svg)](https://github.com/marketplace/actions/arduino_ci)
[![Arduino-lint](https://github.com/RobTillaart/MCP_POT/actions/workflows/arduino-lint.yml/badge.svg)](https://github.com/RobTillaart/MCP_POT/actions/workflows/arduino-lint.yml)
[![JSON check](https://github.com/RobTillaart/MCP_POT/actions/workflows/jsoncheck.yml/badge.svg)](https://github.com/RobTillaart/MCP_POT/actions/workflows/jsoncheck.yml)
[![GitHub issues](https://img.shields.io/github/issues/RobTillaart/MCP_POT.svg)](https://github.com/RobTillaart/MCP_POT/issues)

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/RobTillaart/MCP_POT/blob/master/LICENSE)
[![GitHub release](https://img.shields.io/github/release/RobTillaart/MCP_POT.svg?maxAge=3600)](https://github.com/RobTillaart/MCP_POT/releases)
[![PlatformIO Registry](https://badges.registry.platformio.org/packages/robtillaart/library/MCP_POT.svg)](https://registry.platformio.org/libraries/robtillaart/MCP_POT)


# MCP_POT

Arduino library for MCP41xxx and MCP42xxx SPI based digital potentiometers.


## Description

**Experimental**

The MCP_POT library implements access to MCP digital potentiometers.
These devices have 1 or 2 potentiometers, 10 KΩ, 50 KΩ and 100 KΩ and communicates over SPI.
The library supports both hardware SPI and software SPI.
Since 0.2.1 the SPI speed can be adjusted for both HW and SW SPI.

The library does not support daisy chaining of devices.
This might be implemented in the future.


|  Type      |   KΩ   |  Step Ω  |  Potentiometers  |  Tested  |  Notes  |
|:-----------|:------:|:--------:|:----------------:|:--------:|:--------|
|  MCP41010  |   10   |    39.2  |        1         |     N    |
|  MCP41050  |   50   |    19.6  |        1         |     N    |
|  MCP41100  |  100   |   392.2  |        1         |     N    |
|  MCP42010  |   10   |    39.2  |        2         |     Y    |  daisy chain allowed
|  MCP42050  |   50   |    19.6  |        2         |     N    |
|  MCP42100  |  100   |   392.2  |        2         |     N    |


Current version allows manual override of the hardware SPI clock. 

Alt-234 = Ω


#### 0.2.0 Breaking change

Version 0.2.0 introduced a breaking change to improve handling the SPI dependency.
The user has to call **SPI.begin()** or equivalent before calling **POT.begin()**.
Optionally the user can provide parameters to the **SPI.begin(...)**


#### Related

Mainly other digital potentiometers.

- https://github.com/RobTillaart/AD520x
- https://github.com/RobTillaart/AD524X
- https://github.com/RobTillaart/AD5245
- https://github.com/RobTillaart/AD5144A
- https://github.com/RobTillaart/AD5263
- https://github.com/RobTillaart/X9C10X


## Interface

```cpp
#include "MCP_POT.h"
```

#### Constructors

Base class.

- **MCP_POT(uint8_t select, uint8_t reset, uint8_t shutdown, SPIClassRP2040 \* mySPI = &SPI)** hardware constructor RP2040
- **MCP_POT(uint8_t select, uint8_t reset, uint8_t shutdown, SPIClass \* mySPI = &SPI)** hardware constructor other
- **MCP_POT(uint8_t select, uint8_t reset, uint8_t shutdown, uint8_t dataOut, uint8_t clock)**

The derived classes have same constructors with same parameters as the base class.

- **MCP41010(...)** constructor 1 potentiometer, 10 KΩ
- **MCP41050(...)** constructor 1 potentiometer, 50 KΩ
- **MCP41100(...)** constructor 1 potentiometer, 100 KΩ
- **MCP42010(...)** constructor 2 potentiometer, 10 KΩ
- **MCP42050(...)** constructor 2 potentiometer, 50 KΩ
- **MCP42100(...)** constructor 2 potentiometer, 100 KΩ


- **void begin(uint8_t value = MCP_POT_MIDDLE_VALUE)**
- **void reset(uint8_t value = MCP_POT_MIDDLE_VALUE)**


#### Core

- **bool setValue(uint8_t value)** set both potmeters.
- **bool setValue(uint8_t pm, uint8_t value)**
- **uint8_t getValue(uint8_t pm = 0)**


#### Experimental Ohm interface

The experimental Ohm interface is a wrapper around **setVale()** et al.
It can be used to adjust the value in an SI unit kind of way.
It allows you to set the maximum resistance more exact.

There are however some considerations to be aware of.
- The devices are linear within 1% says datasheet.
- Measuring an MCP 42010, showed the two potentiometers are not 100% identical.
  Measured 10.38 KΩ and 10.37 KΩ, so within accuracy, datasheet < 1%.
- values in Ohm will be rounded, so **getValue()** might not reproduce the parameter
of **setOhm()** or **setValue()** exactly.

- **void setMaxOhm(uint32_t maxOhm)** typical 10000, 50000 or 100000. 
- **uint32_t getMaxOhm()**
- **void setOhm(uint8_t pm, uint32_t ohm)**
- **uint32_t getOhm(uint8_t pm)**

This interface might change into a UNIT based function, in which the unit is not Ohm.
It might be any user defined unit e.g. be flow control, lumen or volume.

(feedback is, as always, welcome)


#### SPI

The default SPI speed used is 1 MHz.

- **void setSPIspeed(uint32_t speed)** sets HW SPI clock in **Hz**.
- **uint32_t getSPIspeed()** gets current HW SPI speed in **Hz**.
- **void setSWSPIdelay(uint16_t del = 0)** adjust SW SPI clock pulse in **us**.
- **uint16_t getSWSPIdelay()** get the current value.


#### Miscellaneous

- **uint8_t pmCount()** returns the number of potmeters.
- **void powerOn()** set SHUTDOWN pin HIGH = device enabled.
- **void powerOff()**  set SHUTDOWN pin LOW = device disabled.
- **bool isPowerOn()** idem.


#### Debug

- **bool usesHWSPI()** returns true if hardware SPI is used.


## About SPI Speed

The SPI code is based upon my MCP_ADC library.

The default SPI speed is reduced to 1 MHz. 
This is the value recommended in the datasheet for 2.7 Volt.

|  Board  |  Voltage  |  Safe   |   Max   |
|:-------:|:---------:|:-------:|:-------:|
|  ESP32  |   2.7V    |  1 MHz  |  4 MHz  |
|  UNO    |   5.0V    |  2 MHz  |  4 MHz  |

For hardware SPI the ESP32 uses the VSPI pins. (see ESP examples).

The example **MCP_POT_performance.ino** measures the performance at different
HW SPI speeds. In a test it showed the MCP42010 still worked at 8 MHz on an UNO.
Note that the use of speeds beyond datasheet specification is at your own risk.


## Daisy Chaining

Not supported yet.

The MCP42xxx series have a **SO = Serial Out** pin which allows to daisy chain the devices.

The devices must share the CS (select) signal, or at least all of them should have 
been selected to forward the bytes that are sent into the chain.
When the CS signal goes HIGH, all devices will simultaneously change to the new values.

Note however that per device at most one potmeter can be set in a daisy chain, 
or both at the same value. (as far as I understand the datasheet on this point).


## Future

#### Must

- improve documentation
- test

#### Should

- investigate and implement daisy chaining of MCP42xxx
- free format wrapper (user can define units is more flex)
  - **setMaxUnit(float mu)**, **setUnit(float unit)**, **getUnit()** etc.
  - still rounding of course, but the user does not need to map any more
  - can be used for Ohm too.

#### Could

- improve SWSPI for AVR 
  (code is under test for MCP23S17)
- percentage interface (another wrapper).


#### Wont


## Support

If you appreciate my libraries, you can support the development and maintenance.
Improve the quality of the libraries by providing issues and Pull Requests, or
donate through PayPal or GitHub sponsors.

Thank you,

