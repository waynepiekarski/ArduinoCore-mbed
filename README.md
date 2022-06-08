# Arduino Core for mbed enabled devices

The repository contains the Arduino APIs and IDE integration files targeting a generic mbed-enabled board

## Installation

### Clone the repository in `$sketchbook/hardware/arduino-git`

```bash
mkdir -p $sketchbook/hardware/arduino-git
cd $sketchbook/hardware/arduino-git
git clone git@github.com:arduino/ArduinoCore-mbed mbed
```

### Clone https://github.com/arduino/ArduinoCore-API into a directory of your choice.

```bash
git clone git@github.com:arduino/ArduinoCore-API
```

### Update the `api` symlink

Create a symlink to `ArduinoCore-API/api` in `$sketchbook/hardware/arduino-git/mbed/cores/arduino`.

### Test things out

Open the Arduino IDE.

You should now see three new targets under the `MBED boards` label.

*This procedure does not automatically install the required ARM compiler toolchain.*

If the toolchain is missing, you'll see errors like this when you try to build for an mbed-os enabled board.:

```
fork/exec /bin/arm-none-eabi-g++: no such file or directory
```
To install ARM build tools, use the `Boards Manager` option in the Arduino IDE to add the `Arduino mbed-enabled Boards` package.

## mbed-os-to-arduino script

The backbone of the packaging process is https://github.com/arduino/ArduinoCore-mbed/blob/master/mbed-os-to-arduino script. It basically compiles a blank mbed-os project for any supported target board, recovering the files that will be needed at compile time and copying them to the right location.

It can be used for a variety of tasks including:

**Recompiling libmbed with source level debug support**

```
cd $sketchbook/hardware/arduino-git/mbed
./mbed-os-to-arduino -a -g PORTENTA_H7_M7:PORTENTA_H7_M7
```

In this case `-a` applies all the patches from `patches` folder into a mainline `mbed-os` tree, and `-g` restores the debug info.

**Selecting a different optimization profile**

```
cd $sketchbook/hardware/arduino-git/mbed
PROFILE=release ./mbed-os-to-arduino -a NANO_RP2040_CONNECT:NANO_RP2040_CONNECT
```

The `PROFILE` environment variable tunes the compilation profiles (defaults to `DEVELOP`). Other available profiles are `DEBUG` and `RELEASE`.

**Selecting a different mbed-os tree**

```
cd $sketchbook/hardware/arduino-git/mbed
./mbed-os-to-arduino -r /path/to/my/mbed-os-fork NICLA_VISION:NICLA_VISION
```

`-r` flag allows using a custom `mbed-os` fork in place of the mainline one; useful during new target development.

**Adding a new target (variant)**

Adding a target is a mostly automatic procedure.

For boards already supported by `mbed-os` , the bare minimum is

```
cd $sketchbook/hardware/arduino-git/mbed
mkdir -p variants/$ALREADY_SUPPORTED_BOARD_NAME/{libs,conf}
./mbed-os-to-arduino $ALREADY_SUPPORTED_BOARD_NAME:$ALREADY_SUPPORTED_BOARD_NAME
# for example, to create a core for LPC546XX
# mkdir -p variants/LPC546XX/{libs,conf}
# ./mbed-os-to-arduino LPC546XX:LPC546XX
```

This will produce almost all the files needed. To complete the port, add the board specifications to `boards.txt` (giving it an unique name) and provide `pins_arduino.h` and `variants.cpp` in `variants/$ALREADY_SUPPORTED_BOARD_NAME` folder.
Feel free to take inspirations from the existing variants :)

For boards not supported by mainline `mbed-os`, the same applies but you should provide the path of your `mbed-os` fork

```
cd $sketchbook/hardware/arduino-git/mbed
mkdir -p variants/$BRAND_NEW_BOARD_NAME/{libs,conf}
./mbed-os-to-arduino -r /path/to/mbed-os/fork/that/supports/new/board $BRAND_NEW_BOARD_NAME:$BRAND_NEW_BOARD_NAME
```

## Using this core as an mbed library

You can use this core as a standard mbed library; all APIs are under `arduino` namespace (so they must be called like `arduino::digitalWrite()` )

The opposite is working as well; from any sketch you can call mbed APIs by prepending `mbed::` namespace.

