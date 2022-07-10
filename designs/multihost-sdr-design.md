# IPMB based SDR Design

Author:
  Velumani T,  [velumanit@hcl](mailto:velumanit@hcl.com)
  Jayashree D, [jayashree-d@hcl](mailto:jayashree-d@hcl.com)

Other contributors: None

Created: July 10, 2022

## Problem Description

SDR is a data record that provides platform management sensor type, locations,
event generation and access information.

A data records that contain information about the type and the number of sensors
in the platform, sensor threshold support, event generation capabilities and
information on what type of readings the sensor provides.

The current version of OpenBMC does not have support for SDR design. This design
will provide the support on how SDR information in each host is received using
IPMB support.

## Requirements

 - Entity manager configuration to support the number of IPMB devices.
 - Dbus-sensors daemon to support SDR sensor information.

## Proposed Design

The below diagram represents the overview for proposed SDR Design.

```

      ENTITY-MANAGER               DBUS-SENSORS


                               +------------------+
                               | +--------------+ |  IPMB 1  +--------+
                               | | Identify SDR | |--------->| Host 1 |
   +------------------+        | | Record Count | |          +--------+
   |                  |        | +--------------+ |
   |   Configuration  |        |                  |  IPMB 2  +--------+
   |      file to     |        | +--------------+ |--------->| Host 2 |
   |   identify the   |------->| | Reserve SDR  | |          +--------+
   |     number of    |        | | Repository   | |             ....
   |   IPMB Devices   |        | +--------------+ |             ....
   |                  |        |                  |             .... 
   +------------------+        | +--------------+ |  IPMB N  +--------+
                               | | Get SDR Data | |--------->| Host N |
                               | +--------------+ |          +--------+
                               +------------------+

```

This document proposes a design for multi-host SDR implementation.

 - Need to create a configuration file to detect the number of IPMB Devices.

 - Once the IPMB Device is detected, SDR Repository information and the record
   count of the sensor will be identified for each host using IPMB SendRequest.

 - Reservation ID for SDR sensor will also be received after the record count
   using IPMB support.

 - Data packet will be framed based on the reservation ID to get the full
   information of each sensor like sensor name, sensor type, sensor unit,
   threshold values, sensor unique number using IPMB.

 - After all the information is read, sensor details will be displayed in the
   dbus path under IpmbSensor service.

**Example**

```
ENTITY-MANAGER :

     busctl tree xyz.openbmc_project.EntityManager
     `-/xyz
       `-/xyz/openbmc_project
         |-/xyz/openbmc_project/EntityManager
         `-/xyz/openbmc_project/inventory
          `-/xyz/openbmc_project/inventory/system
            `-/xyz/openbmc_project/inventory/system/board
              |-/xyz/openbmc_project/inventory/system/board/<ProbeName>
              | |-/xyz/openbmc_project/inventory/system/board/<ProbeName>/1_IpmbDevice
                               ............
                               ............
              |-/xyz/openbmc_project/inventory/system/board/<ProbeName>
              | |-/xyz/openbmc_project/inventory/system/board/<ProbeName>/N_IpmbDevice

DBUS-SENSORS :

     busctl tree xyz.openbmc_project.IpmbSensor
     `-/xyz
       `-/xyz/openbmc_project
         `-/xyz/openbmc_project/sensors
           `-/xyz/openbmc_project/sensors/current
             `-/xyz/openbmc_project/sensors/current/sensor1
                         ............
                         ............
           `-/xyz/openbmc_project/sensors/power
             `-/xyz/openbmc_project/sensors/power/sensor1
                         ............
                         ............
           `-/xyz/openbmc_project/sensors/temperature
             `-/xyz/openbmc_project/sensors/temperature/sensor1
                         ............
                         ............
           `-/xyz/openbmc_project/sensors/voltage
             `-/xyz/openbmc_project/sensors/voltage/sensor1
```

Following modules will be updated for this implementation.

 - Entity-Manager
 - Dbus-Sensors

## Entity-Manager

A configuration file is created to probe the IPMB FRU devices to detect the
number of the IPMB support present and dbus object path & interface is
created under entity-manager service.

## Dbus-Sensors

**IpmbSensor**

After the interface is created for IPMBDevice in entity-manager, dbus-signal
will be notified to IpmbSensor. For each config, based on the bus index for
each host, SDR Repository information will be received using IPMB.

This information will provide the number of SDR records and SDR Reservation ID
will be received using IPMB support. Then, data packet will be framed based on
reservation ID to get the full information of each sensor using IPMB.

Once all the sensor information is read, each data will be processed and
displayed in dbus path under IpmbSensor service.

## Impacts

This design does not have any impacts on the existing changes as the code is
implemented based on the IPMB Devices detected in entity-manager.
