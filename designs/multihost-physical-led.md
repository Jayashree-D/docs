# Multi-Host Physical LED Design Support

Author: 
  Velumani T,  [velumanit@hcl](mailto:velumanit@hcl.com)
  Jayashree D, [jayashree-d@hcl](mailto:jayashree-d@hcl.com)

Other contributors: None

Created: July 04, 2022

## Problem Description

In the existing phosphor-led-sysfs design, it exposes one service per LED
configuration in device tree. Hence, multiple services will be created for
multiple GPIO pins configured for LEDs.

There are cases where multiple LEDs were configured in the device tree for each
host and needs to be paired as a group of LEDs for the specified host in the
multi host platform.

Based on the current design, pairing groups will be difficult under multiple
services. To abstract this method and also to create LEDs under a single
service, a new implementation is proposed.

## Background and References

The below diagram represents the overview for proposed physical LED design.

```

    DEVICE TREE                  BMC                     PHOSPHOR-LED-SYSFS

   +------------+    Pin 1    +----------+              +--------------------+
   |            |------------>|          |              |                    |
   | GPIO PIN 1 |    Pin 2    |   UDEV   | Dbus Signals |  Method to handle  |
   |            |------------>|  Events  |------------->|     udev event     |
   | GPIO PIN 2 |     ...     |          |              |                    |
   |            |     ...     | (To Save |              +--------------------+
   | GPIO PIN 3 |    Pin N    |   LED    |                         |
   |            |------------>|  names)  |                         |
   | GPIO PIN 4 |             +----------+                         V
   |            |                                 +------------------------------+
   |   .....    |                                 |                              |
   |   .....    |      +------------------+       |  Service :                   |
   |   .....    |      |                  |       |                              |
   |            |      |  LED Controller  |       |  /xyz/openbmc_project/<led1> |
   | GPIO PIN N |      |     Service      |       |  /xyz/openbmc_project/<led2> |
   |            |      |      (xyz.       |------>|  /xyz/openbmc_project/<led3> |
   +------------+      |  openbmc_project.|       |         ........             |
                       |  LED.Controller) |       |         ........             |
                       |                  |       |  /xyz/openbmc_project/<ledN> |
                       +------------------+       |                              |
                                                  +------------------------------+

```

Since LEDS cannot pair a group under multiple services, so it is modified to
single service and groups can also be formed as per specified host's LEDs.

## Requirements

 - Udev rules to receive LED events.
 - Support for Dbus signal to send LED events to phosphor-led-sysfs method.
 - Provide a single systemd service for phosphor-led-sysfs application.

## Proposed Design

This document proposes a new design for physical LED implementation.

 - Physical Leds are defined in the device tree under "leds" section.

 - Corresponding GPIO pin are defined for the physical LEDs.

 - Phosphor-led-sysfs will have a single systemd service
   (xyz.openbmc_project.led.controller.service) running by default at
   system startup.

 - "udev rules" are handled to monitor the physical LEDs events.

 - Once the udev event is initialized for the LED, it will send those LED name
   using the dbus signal in udev instead of triggering systemd service.

 - A dbus method call will be exposed from the service. udev will notify the
   LEDs detected in the driver to the method call in phosphor-led-sysfs daemon.

**Example**

```
     busctl tree xyz.openbmc_project.LED.Controller
     `-/xyz
       `-/xyz/openbmc_project
         `-/xyz/openbmc_project/led
           `-/xyz/openbmc_project/led/physical
             `-/xyz/openbmc_project/led/physical/led1
             `-/xyz/openbmc_project/led/physical/led2
                         ............
                         ............
             `-/xyz/openbmc_project/led/physical/ledN
```

Following modules will be updated for this implementation.

 - OpenBMC - meta-phosphor
 - Phosphor-led-sysfs

## OpenBMC - meta-phosphor

udev rules is created for physical LEDs and udev events will be created when
the LED GPIO pins are configured in the device tree.

udev events will receive the LED names which are configured in the device tree
as arguments and send those names to a method in phosphor-led-sysfs application
using dbus signal, whenever udev events is created.

## Phosphor-led-sysfs

Phosphor-led-sysfs will have a single systemd service which will be started
running at the system startup. Once the application started, it will invoke a
method to handle the LED udev events which are received.

Based on the LED udev events, all the LEDs name will be retrieved and dbus path
will be created for all the LEDs under single systemd service.

**D-Bus Objects**

```
   Service        xyz.openbmc_project.LED.Controller

   Object Path    /xyz/openbmc_project/led/physical/ledN

   Interface      xyz.openbmc_project.LED.Physical
```

## Impacts

These changes are under phosphor-led-sysfs design and this diplays all the
physical LED dbus path under single systemd service instead of multiple
services. This will not impact the single host physical LED design.

## Testing

The proposed design can be tested in both single and multiple hosts platform.
