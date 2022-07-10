# IPMB based SDR Design

Author:
  Velumani T,  [velumanit@hcl](mailto:velumanit@hcl.com)
  Jayashree D, [jayashree-d@hcl](mailto:jayashree-d@hcl.com)

Other contributors: None

Created: July 08, 2022

## Problem Description

SDR is a data record that provides platform management sensor type, locations,
event generation and access information.

A data records that contain information about the type and the number of sensors
in the platform, sensor threshold support, event generation capabilities and
information on what type of readings the sensor provides.

The current version of OpenBMC does not have support for SDR design. Therefore,
this design will describe the sensor information in each host to the BMC using
IPMB support.

## Background and References

```



```

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


