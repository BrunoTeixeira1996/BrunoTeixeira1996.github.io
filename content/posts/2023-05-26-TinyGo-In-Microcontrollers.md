---
title: "TinyGo in Microcontrollers"
date: 2023-05-26
draft: false
toc: true
tags:
  - Golang
---

## Index

- [Introduction](#introduction)
- [TinyGo](#tinygo)
- [XIAO](#xiao)
- [Flashing](#flashing)
- [Reset](#reset)
- [Resources](#resources)

## Introduction

Recently (at work) I had to flash a microcontroller to simulate a keyboard. The problem was that I didn't knew anything about microcontrollers (still don't) or even how could I flash those.

At the time I was using the [Digispark](http://digistump.com/products/1) microcontroller since it was cheap and it was user friendly (with an external case, the device looks like a normal usb). Then a co-worker helped me on the flashing process and for that we used [Arduino IDE](https://www.arduino.cc/en/software).

This process was a bit overwhelm because I had to run multiple scripts in order to convert keyboard presses into arduino language (because I wanted to run Rubber Ducky scripts), [however, the way this can be done is well documented on the internet](https://null-byte.wonderhowto.com/how-to/run-usb-rubber-ducky-scripts-super-inexpensive-digispark-board-0198484/).

After this, I decided to take a look at other alternatives since Arduino IDE was sometimes buggy and not user friendly at all. And with that I've found [TinyGo](https://tinygo.org/).

## TinyGo

Looking at the TinyGo website: _TinyGo brings the Go programming language to embedded systems and to the modern web by creating a new compiler based on LLVM_.
Basically, you can compile and run Go code inside microcontrollers by using Tinygo.

After some hours of digging in the documentation I've found that [they support the USB HID for some devices](https://github.com/tinygo-org/tinygo/issues/1118) and in order to simulate keyboard presses I needed the [USB HID](https://en.wikipedia.org/wiki/USB_human_interface_device_class).

However, the device I had was a Digispark and according to one of the TinyGo maintainers, this device does not have USB hardware so it needs to be [bit-banged](https://en.wikipedia.org/wiki/Bit_banging).

![image](https://github.com/BrunoTeixeira1996/go-microcontrollers/assets/12052283/a5684a13-6004-4fc7-bb5c-5f83ce42ed57)

After this, I realised that I needed to choose another microcontroller (until they don't support Digispark). 

Looking at the [release notes](https://github.com/tinygo-org/tinygo/releases) for TinyGo I've found what devices they support for USB HID.

![image](https://github.com/BrunoTeixeira1996/go-microcontrollers/assets/12052283/5f4a822c-2d93-43df-b21e-acf522e922f7)

`samd21` looked promising, so I went out and bought a [Seeed XIAO samd21](https://www.seeedstudio.com/Seeeduino-XIAO-Arduino-Microcontroller-SAMD21-Cortex-M0+-p-4426.html).


## XIAO

XIAO SAMD21 is an ultra-small universal development board, supporting Arduino / Micropython / CircuitPython and much more.

It operates up until 48MHz and it has 32KB of SRAM and 256KB of onboard flash memory.

In terms of development interfaces:
- 11x analog / 11x digital Pins
- 1x I2C interface
- 1x UART port
- 1 SPI port

![image](https://github.com/BrunoTeixeira1996/go-microcontrollers/assets/12052283/149517f2-9bfd-452e-bb16-645491ddd6ed)


## Flashing

### XIAO

Looking at this [issue](https://github.com/tinygo-org/tinygo/issues/3639) we can see that TinyGo flash does not work on Seeed XIAO because as recently as Dec 2022, the Seeed Studio XIAO M0 dev boards shipped with a bootloader from Nov 2019: UF2 Bootloader v3.7.0-33-g90ff611-dirty SFHWRO. 

Knowing this, I had to find the new MSD (Mass Storage Device). For that, I plugged the device into my computer, opened the device in my file manager and checked the files present in it. 
After getting the new MSD name, I've created a new target (in `/usr/local/lib/tinygo/targets/xiao-new.json`) as shown below.

```console
{
    "inherits": ["atsamd21g18a"],
    "build-tags": ["xiao"],
    "serial": "usb",
    "serial-port": ["2886:802f"],
    "flash-1200-bps-reset": "true",
    "flash-method": "msd",
    "msd-volume-name": "Seeed XIAO",
    "msd-firmware-name": "firmware.uf2"
}
```

After this, the flash command still didn't worked (at least for me) so in order to fix this, everytime I wanted to upload the new firmware to the microcontroller I had to compile the program and move the `.uf2` to the XIAO device

The following shows an example to emulate a keyboard by opening xcalc on my device (using i3wm with dmenu).

```go
package main

import (
	"machine/usb/hid/keyboard"
)

/*
WINDOWS KEY + ENTER (opens dmenu)
WRITE xcalc
ENTER
*/
func Openxcalc() {	
	kb := keyboard.Port()
	runs := 1
	for {
		if runs == 2 {
			break
		}
		runs +=1

		time.Sleep(1000 * time.Millisecond)

		kb.Down(keyboard.KeyModifierGUI)
		kb.Down(keyboard.KeyEnter)
		kb.Release()

		time.Sleep(1000 * time.Millisecond)

		kb.Write([]byte("xcalc"))

		time.Sleep(1000 * time.Millisecond)

		kb.Press(keyboard.KeyEnter)
	}
}
```

```bash
$ tinygo build -o firmware.uf2 -target=xiao-new main.go && sudo mkdir -p /media/xiao && sudo mount /dev/sda /media/xiao && sudo cp firmware.uf2 /media/xiao
```

<video src="https://github.com/BrunoTeixeira1996/go-microcontrollers/assets/12052283/5649e74e-2488-4e40-81b3-76c5548bc391" controls="controls" style="max-width: 730px;"> </video>

### Digispark

In order to show how TinyGo works in Digispark, Ill demonstrate a flashing led in the Digispark device.

For that, download the latest release of micronucleus (https://github.com/micronucleus/micronucleus/releases) and perform the following steps:

```bash
$ unzip ~/Downloads/micronucleus-cli-master-882e7b4a
$ sudo cp micronucleus-cli-master-882e7b4a/micronucleus /usr/local/bin/micronucleus
$ export PATH=$PATH:/usr/local/bin/micronucleus
$ tinygo flash -target=digispark digispark/digispark.go
INSERT USB STICK
```

The following code is the flashing led provided by TinyGo.

```go
package main


import (
	"machine"
)

// Blinking LEDs
func BlinkingLEDs() {
	led := machine.LED
	led.Configure(machine.PinConfig{Mode: machine.PinOutput})
	for {
		led.Low()
	    time.Sleep(1000 * time.Millisecond)

		led.High()
	    time.Sleep(1000 * time.Millisecond)
	}
}
```

<video src="https://github.com/BrunoTeixeira1996/go-microcontrollers/assets/12052283/8eb0e0e1-4bf9-480c-a5b0-bdf5de32eda0" controls="controls" style="max-width: 730px;"> </video>

## Reset

There are two simple techniques to reset the devices, after flashing them.

The first works in XIAO and the goal is to double tap the RST pins like is shown [here](https://wiki.seeedstudio.com/Seeeduino-XIAO/#enter-bootloader-mode).

To reset the Digispark board, we can use the same `flash` command but now with an empty `for loop`.



## Resources

- https://tinygo.org/
- https://circuitdigest.com/article/introduction-to-bit-banging-spi-communication-in-arduino-via-bit-banging
- https://wiki.seeedstudio.com/Seeeduino-XIAO/
- https://github.com/BrunoTeixeira1996/go-microcontrollers/tree/main/programs
