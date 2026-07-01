---
layout: post
title: "Getting Started with the M5Stack Atom Matrix with MicroPython"
date: 2026-06-18
categories: python programming
---
I've accumulated more ESP32 boards than I'd like to admit. Some end up forgotten in a drawer after a weekend project, while others become regular fixtures on my workbench. The M5Stack Atom Matrix falls into the second category.

It's tiny, surprisingly capable, and just plain fun to work with. It has an ESP32, Wi-Fi, Bluetooth, a programmable button, and a 5x5 RGB LED matrix packed into a package that's barely larger than a USB flash drive.

The best part is that you can skip the Arduino toolchain entirely and use MicroPython. If you already know a little Python, you'll be writing code on the board within minutes.

## What You'll Need

For this guide I'm using an M5Stack Atom Matrix and a USB-C cable that supports data transfer. You'll also need:

* Python 3 installed on your computer
* `esptool` for flashing firmware
* `mpremote` for talking to the board

If you don't already have the tools installed:

```bash
pip install esptool mpremote
```

## Flashing MicroPython

The Atom Matrix uses the ESP32, so grab the latest generic ESP32 firmware from the official MicroPython downloads page.

Once you've downloaded the firmware, plug the board into your computer.

It's a good idea to erase the flash first:

```bash
esptool.py --chip esp32 erase_flash
```

Then write the firmware:

```bash
esptool.py --chip esp32 --baud 460800 write_flash -z 0x1000 esp32.bin
```

Replace `esp32.bin` with whatever the downloaded firmware file is called.

The flashing process only takes a minute or two.

## Connecting to the REPL

One of my favorite things about MicroPython is the interactive REPL. Instead of compiling, uploading, and waiting, you can just type Python directly into the device.

Connect with:

```bash
mpremote connect auto
```

If everything worked, you'll see:

```python
>>>
```

Now you're talking directly to the ESP32.

Try something simple:

```python
print("Hello from the Atom Matrix!")
```

If the board prints the message back, you're ready to start experimenting.

## Uploading Programs

While the REPL is great for testing ideas, eventually you'll want your code to survive a reboot.

Create a file named `main.py` and copy it to the device:

```bash
mpremote fs cp main.py :
```

MicroPython automatically runs `main.py` every time the board starts.

You can also inspect the filesystem:

```bash
mpremote fs ls
```

Or run a script without copying it permanently:

```bash
mpremote run test.py
```

I use this workflow constantly because it feels almost instant.

## What About the LED Matrix?

The 5x5 RGB matrix is one of the reasons people buy the Atom Matrix, but it's also the one part that trips up new users.

Unlike the GPIO pins, the LEDs aren't controlled directly through the standard MicroPython library. You'll either need to use a NeoPixel-compatible driver or install one of the community libraries written specifically for the Atom Matrix.

The good news is that once it's working, you can create everything from status indicators to scrolling text and tiny animations.

## A Few Things I Learned

After spending some time with the board, a few habits have made development much easier.

Use the REPL to test ideas before writing a full program. It saves a surprising amount of time.

Keep Wi-Fi credentials in a separate configuration file instead of scattering them throughout your project.

Don't panic if the serial port disappears after flashing. Unplug the board, plug it back in, and reconnect with `mpremote`.

And finally, keep a data-capable USB-C cable nearby. A surprising number of USB-C cables only provide power, and they will waste far more of your time than the ESP32 ever will.

## Final Thoughts

The M5Stack Atom Matrix is one of those boards that's easy to overlook because of its size. Once you start using it, though, it's hard not to appreciate how much hardware M5Stack managed to fit into such a small package.

MicroPython makes the experience even better. Instead of waiting for builds and uploads, you can experiment interactively, iterate quickly, and spend your time solving problems instead of fighting your toolchain.

If you're looking for a compact ESP32 board that's enjoyable to hack on, the Atom Matrix is a great place to start.
