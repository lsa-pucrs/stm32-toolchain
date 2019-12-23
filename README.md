# STM32 Toolchain

Use this "toolchain" when compiling and uploading programs to STM32 ARM devices.
This repository was cobbled together from several tutorials and other repos (links below), as well as some
research to find a flasher compatible with our chips ([CS32F103C8T6](https://pt.aliexpress.com/item/32525208361.html?spm=a2g0s.9042311.0.0.27424c4deFqZ4c)).

Below are general instructions for installation and usage. This repository contains all the necessary files, however it might
be prudent to replace them with newer versions when available.

## Dependancies

* [ARM EABI GCC compiler](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads) (NOTE: Must be "arm-none" version)
* [Open source ARM Cortex-M microcontroller library](http://libopencm3.org/) 
* build-essentials
* git
* make

## Installation

* Firstly, you must clone this repository:

```
$ git clone https://github.com/lsa-pucrs/stm32-toolchain $HOME/stm32-toolchain
```

* For convenience, let's define a variable to indicate the path of the toolchain's root folder:

```
$ export STM32TOOLCHAIN=$HOME/stm32-toolchain
```

* Go into the repository's root folder and run git's submodule utility to initialize and update the external repositories:

```
$ cd $STM32TOOLCHAIN
$ git submodule update --init
```

### ARM EABI GCC

* Go into the repository's root folder and move the "gcc-arm-none-eabi" folder to a system-wide location (e.g., /opt):

```
$ cd $STM32TOOLCHAIN
$ sudo mv gcc-arm-none-eabi /opt
```

* Include the binary path in your PATH environmental variable

```
$ export PATH=$PATH:/opt/gcc-arm-none-eabi/bin
```

* (OPTIONAL) In Ubuntu, you can make the binaries always visible by updating the .bashrc file:

```
$ echo "export PATH=\$PATH:/opt/gcc-arm-none-eabi/bin" >> .bashrc
```

### OpenCM Library

[Warren Gay](https://github.com/ve3wwg) has created a highly convenient installation template for the OpenCM library and ST-Link utility, although the latter
has no support for the CS32F103C8T6.

* Go into Warren's template repo file:

```
$ cd $STM32TOOLCHAIN/stm32f103c8t6
```

* Run git's submodule update to download the newest version of the OpenCM library:

```
$ git submodule update --init
```

* Run "make" to compile the library. 

At this point, you should be able to write and build programs for your STM32 board, although you need the STLink utility to upload them.

### STLink

To upload programs to the board, you need a bootloader, such as the STLink:

![](https://http2.mlstatic.com/programador-stm32-stm8-stlink-st-link-v2-mini-D_NQ_NP_867041-MLB30915342638_052019-F.jpg)

This bootloader interfaces with the ST-Link utilities. As we are using the CS32F103C8T6 chip, we will use a flasher utility compatible with this board (from [this](https://github.com/texane/stlink) repository, support added in [this](https://github.com/texane/stlink/pull/757/commits/84a89cb98e5169fc712133716c0f5ab15a3904a5) commit).

* Go into the "stlink" folder and run "make" to compile the utilities:

```
$ cd $STM32TOOLCHAIN/stlink
$ make
```

* Go into the "build/Release" folder and run "make install" (as root) to install the utilities:

```
$ cd $STM32TOOLCHAIN/stlink/build/Release
$ sudo make install
```

* (OPTIONAL) To check if they are working, run the st-info utility:

```
$ st-info --version
```

If you get a "command not found" error, check if the "st-info" file is present in the "/usr/local/bin" folder.
To see if your STLink bootloader works, run "st-info --probe" as root. You should see some information on your device as output.

## Compiling and running a test program

Warren's OpenCM template includes a "miniblink" program, which blinks an LED on and off.
To compile the program, go into the "miniblink" folder and run "make":

```
$ cd $STM32TOOLCHAIN/stm32f103c8t6/miniblink
$ make
```

This will cross-compile the code for the STM32 and produce two files: "miniblink.elf" and "miniblink.bin".
To upload the program to the board, run the "st-flash" utility as root:

```
$ sudo st-flash write miniblink.bin 0x8000000
```

Alternatively, you can run "make flash" as root. If all went well, you should see your blinking led on the STM32 board.

## Acknowledgements

* Olayiwola Ayinde's "[Programming STM32 on Linux](https://medium.com/@olayiwolaayinde/programming-stm32-on-linux-d6a6ee7a8d8d)" tutorial
* Warren Guy's "[libopencm3 and FreeRTOS](https://github.com/ve3wwg/stm32f103c8t6)" template repository

## Troubleshooting

* Communication problems when flashing (e.g., LIBUSB timeout error)

You may need to update your STLink firmware. Follow [this](http://www.emcu.eu/how-to-update-the-st-link-fw-under-linux/) tutorial on how to do so.
