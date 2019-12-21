# Introduction

Seesaw is an open source microcontroller friend for other chips. It provides a
variety of capabilities such as UART, ADC, DAC, extra GPIO, etc. to chips that don't have them.

# Interfacing
- [Arduino](https://github.com/adafruit/Adafruit_Seesaw)
- [CircuitPython and Python](https://github.com/adafruit/Adafruit_CircuitPython_seesaw)
- [Legacy Python](https://github.com/adafruit/Adafruit_Python_seesaw)

## Build

### Requirements

* `make` and a Unix environment
* `python` in path
* `arm-none-eabi-gcc` and `arm-none-eabi-g++` in the path
    - For Ubuntu 18.04, follow these [setup instructions](https://learn.adafruit.com/building-circuitpython/linux#install-build-tools-on-ubuntu-2-2)
### Build commands

The default board is `debug`. You can build a different one using:

```
make BOARD=samd09breakout
```

add `DEBUG=1` to the `make` command to enable UART debug logging. The default UART TX pin is `PA04` and the default baud rate is `115200`. Note that there may not be enough available flash to enable UART debug loggnig on smaller chips such as the SAMD09. Also note that the I2C Slave interface will not be able to accept commands as fast while UART debug logging is enabled and use of the flow control pin is required.

### Generating a new configuration

A new configuration can be generated by running:
```
./generate.py
```
from the repository root directory and then answering the series of prompts that follows. Note that this script currently only supports SAMD09-based configurations.

### Adding new modules

The following is a brief overview of files that need to be changed when a new module is added.
Each module should be added in the form of a C++ class. `.h` files go in the `inculde` folder, and `.cpp` files go in the `source` folder. Any peripheral drivers should be added to the `bsp` folder.

add the new source files to the `Makefile`

follow the format in `source/main.cpp` to include the new class header file, conditionally statically instantiate the new module, and conditionally start the new module in the main function.

add an enum value for the new module as well as a unique priority in `include/hsm_id.h`

add the virtual module base as well as any new virtual registers to `include/RegisterMap.h`

add any unique events (and event class definitions if the events take parameters) for the new module to `include/event.h`
NOTE: make sure to add the event name strings to the correct spot in `source/event.cpp` for UART logging.

add any configuration definitions for the new modules in `include/SeesawConfig.h` and add any used pins to the `CONFIG_GPIO_MASK` definition.

conditionally subscribe the `System` object to the new START, START_CFM, STOP, and STOP_CFM events in `System::InitialPseudoState` in `souce/System.cpp` and make sure the `System` object starts and stops the new module at the desired time.

lastly, route any desired read and write host commands to the new module in the switch statement in the `DELEGATE_PROCESS_COMMAND` event in `source/Delegate.cpp`

# Future Work

Much of the boilerplate work listed above could be automated away using the preprocessor and other build tools so only the new module files would need to be edited.

# Contributing

Contributions are welcome! Please read our [Code of Conduct](https://github.com/adafruit/seesaw/blob/master/CODE_OF_CONDUCT.md) before contributing to help this project stay welcoming.
