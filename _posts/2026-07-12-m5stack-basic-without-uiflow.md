---
layout: post
title: "Using the M5Stack Basic with UIFlow Firmware Without the Cloud"
date: 2026-07-12
categories: python programming
---

When you first power on an M5Stack Basic, the firmware gives a strong impression that the device is built around the UIFlow cloud environment. It starts with a setup menu, asks for Wi-Fi credentials and expects an API key before it settles into normal operation.

That first impression can be misleading.

Underneath the UIFlow experience is a full MicroPython environment with drivers already included for the display, buttons, speaker, SD card and other hardware. You can write MicroPython applications directly on the device without compiling firmware or installing additional libraries.

The part that makes UIFlow feel cloud dependent is mostly the startup process. Once that layer is understood, the firmware becomes much more interesting. You can keep the hardware support while ignoring the browser based workflow entirely.

For local development using a serial terminal or an editor such as Thonny, the UIFlow firmware turns out to be a convenient middle ground between stock MicroPython and a fully managed ecosystem.

## Looking Beneath UIFlow

The key to understanding the firmware is `boot.py`.

Like other MicroPython boards, the M5Stack Basic runs `boot.py` before executing `main.py`. The supplied version is designed around the UIFlow workflow. It reads configuration values from the ESP32's non volatile storage, decides how the board should start, handles Wi-Fi setup and communicates with the UIFlow service.

It also checks for over the air updates and can replace `main.py` with a downloaded version.

For standalone MicroPython development, most of this is unnecessary.

The important discovery is that the hardware support is not tied to these cloud features. The LCD, buttons, speaker, SD card and other peripherals are already supported by the firmware. The online functionality is mostly a startup convenience layer that can be bypassed.

## Understanding the Filesystem

The UIFlow filesystem is divided into two main areas.

The `/flash` directory contains application files including `boot.py`, `main.py` and anything else you create. This is where normal development happens.

The `/system` directory contains files included with the firmware including fonts, libraries and interface assets.

One directory stands out when exploring the default installation: `/system/basic`.

It contains JPEG images used by the UIFlow startup screens and example projects. If you are not using the browser based UIFlow environment these files mostly consume flash storage without providing much value.

Removing `/system/basic` frees space for your own applications. It can also be replaced with your own images, fonts or project assets if your application benefits from storing resources directly on the device.

## Controlling Startup With NVS

The UIFlow startup behavior is controlled by values stored in NVS, the ESP32's built in Non Volatile Storage.

Unlike files under `/flash`, NVS is managed by the ESP32 firmware itself. It is a small key value database designed to survive resets and power loss.

UIFlow stores its configuration in an NVS namespace called `uiflow`. This includes settings such as Wi-Fi credentials, NTP servers, the UIFlow server address and the startup mode.

The most useful entry is `boot_option`.

```
0 - Run main.py directly
1 - Show the startup menu and connect to Wi-Fi
2 - Connect to Wi-Fi without showing the startup menu
```

You can inspect or modify this value directly from MicroPython.

```python
import esp32 nvs = esp32.NVS("uiflow") 
print(nvs.get_u8("boot_option")) 
nvs.set_u8("boot_option", 0) 
nvs.commit()
```

Changing `boot_option` to `0` tells the supplied `boot.py` to skip the UIFlow synchronization process and start your application directly.

This is a good example of how loosely coupled the cloud features are. There is no need to rebuild firmware or modify binaries. A single configuration value changes the startup behavior.

## Exploring the NVS Partition

Since NVS is separate from the filesystem it has its own flash partition.

The ESP32 partition table reveals where it is stored:

`python -m esptool read-flash 0x8000 0x1000 partitions.bin`

Partition entries are 32 bytes long. A typical NVS entry looks like this:

```
50aa 0201 9000 0000 6000 0000 766e 0073
```

Decoded:

```
Offset : 0x9000
Size   : 0x6000
Label  : nvs
```

The labels are stored as plain ASCII, making them easy to find when examining the binary.

The entire partition can then be dumped:

`python -m esptool read-flash 0x9000 0x6000 nvs.bin`

A quick look with `strings` reveals many useful details:

`strings -tx nvs.bin`

Typical UIFlow entries include:

```
boot_option server
ssid0 pswd0 ssid1 pswd1 ssid2 pswd2
ntp0 ntp1 ntp2 tz # as in: Time Zone
```

For a complete decode, the NVS tools included with ESP-IDF can interpret the data structures and display namespaces, keys and values.

Once the keys are known, they can also be inspected directly from MicroPython.

```python
import esp32 
nvs = esp32.NVS("uiflow")
keys = [ 
    ("boot_option", "u8"), ("server", "str"), ("ssid0", "str"),
    ("pswd0", "str"), ("ssid1", "str"), ("pswd1", "str"),
    ("ssid2", "str"), ("pswd2", "str"), ("ntp0", "str"), 
    ("ntp1", "str"), ("ntp2", "str"), ("tz", "str")
    ] 

for key, typ in keys: 
    try: 
        if typ == "u8": value = nvs.get_u8(key) 
        elif typ == "i32": value = nvs.get_i32(key) 
        elif typ == "str": value = nvs.get_str(key) 
        elif typ == "blob": value = nvs.get_blob(key) 
        else: value = "<unknown type>" print(f"{key:12} = {value!r}") 

    except Exception as e: print(f"{key:12} : {e}")
```

This is also a useful reminder that anything stored in NVS should be treated carefully. Code running on the MicroPython interpreter can access these values, including Wi-Fi credentials and other configuration data.

## The Simple Solution

After understanding the startup process, the easiest approach is surprisingly simple.

Replace the supplied `boot.py` with a minimal version.

```python
# boot.py 
print("Booting...")
```

Once `boot.py` finishes, MicroPython automatically runs `main.py`.

The board now starts directly into your application without showing the UIFlow menu, connecting to Wi-Fi or contacting the cloud service.

## Why Keep UIFlow Firmware?

When I started looking into the M5Stack Basic, the obvious solution seemed to be replacing UIFlow with standard ESP32 MicroPython firmware.

That works, but it also means giving up the convenience of the included drivers.

The UIFlow firmware already knows how to talk to the LCD, buttons, IMU, speaker and other peripherals. There is no searching for libraries, configuring drivers or building custom firmware just to get the hardware working.

Once the cloud startup behavior is removed, those drivers become the main reason to keep the firmware.

The result is a lightweight local development environment with excellent hardware support.

## Working Offline

After simplifying the startup process, the M5Stack Basic behaves like a normal MicroPython board.

You can:

* Edit files over a serial connection.
* Run applications from `main.py`.
* Use the SD card locally.
* Develop with or without Wi-Fi.
* Ignore UIFlow API keys completely.

The board boots faster, avoids unnecessary network activity and gives full control back to the developer.

## Final Thoughts

The interesting thing about UIFlow firmware is that its cloud features are only part of the story.

The default startup experience makes the device look tied to an online service, but underneath is a capable MicroPython platform with a large collection of hardware support already included.

A minimal `boot.py` is enough to remove the parts you do not need while keeping the parts that make the firmware useful.

Deleting unused UIFlow artwork from `/system/basic` provides additional storage for your own projects.

What started as an attempt to replace the firmware becomes a much better solution: keep the firmware, remove the assumptions. The result is a fast, local and capable MicroPython environment that makes the M5Stack Basic an even more flexible development platform.
