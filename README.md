# vicHUD
Android/Arduino based Vehicle Heads-Up-Display for vehicles after 2002 using OBD-II ports and CAN protocol

This is a base README file for the project. Overall plan for the vicHUD:

## Introduction

The purpose of this project is to produce a Heads-Up Display (HUD) to accurately, safely, fashionably, and rapidly display useful information to a driver about their vehicle and their current trip. Vehicle manufacturers today already have implimented HUDs as part of their "cockpit" experiences, believing that drivers can check important information such as speed and direction without taking their eyes off of the road. Recent HUD units display everything from GPS guidance to augmented reality assisted lane switching. While not all of these features will be initially realistic for this project, it should still make the basics of a vehicle HUD available to those without newer expensive cars. Advanced users should also be able to customize their interface or even build their own features as they see fit. Eventually, we hope that the project will attract open source developers interested in expanding vehicle compatibility, or frankly, whatever they want.

## Features

As of March 2020, the HUD will support:

- Vehicle Speed
- Vehicle Direction
- Vehicle RPM
- Current Time and Date
- Blind Spot Detection
- Read/Front Parking Camera
- Entertainment System Integration

## Data Flow Concept

In 1996, the US standardized steering wheel-side OBD-II ports for the convenience of auto repair shops and manufacturers. While this aided the vehicle diagnosis process, vehicle manufacturers did not agree on a standard pin configuration until the US passed another law in 2002. After 2002, all US vehicles (and most vehicles around the world) now support the CAN Bus configuration. 

All physical layer attributes are identical. The port itself is a 9-pin configuration with only 4 in use:

PIN 2 - CAN LOW
PIN 3 - GROUND
PIN 7 - CAN HIGH
PIN 9 - CAN V+ 

The CAN Bus links with every Electronic Control Unit (ECU) in the vehicle, and as a general rule, the newer the vehicle, the more ECUs it will have, and the more information will be available through the bus. 

In lower level layers, there is still some variation in the CAN Bus standard. SAE J2284 and J1979 is primarily used in passenger vehicles while MilCAN is specifically designed for military vehicles. As vehicle instruments improve over time, new protocols will be made to support greater information flow and more specific use cases (e.g. low layer protocols like ISO 16845 are updated annually).

For the HUD display purpose, the CAN Bus protocol is much simpler. We capture and deliver information in frames of data, described below:

1 Start of Frame Bit
29 CAN-ID Bits (Extended Identifier)
1 Remote Transmission Request Bit
6 Control Bits
0-64 Data Bits
16 Cycle Redundancy Check Bits
2 ACK Bits
7 End of Frame Bits

Raw CAN Bus data shows up as a flood of bits, but with some sniffing and parsing, it is possible to decode it into a useable form. To connect to the OBD-II port, an ELM327 interpreter is necessary to convert analogue information in HEX. The message structure of a message looks like this:

| 11 Bit Indentifier | 64 Bit Data |

The 64 bit data portion contains the bread and butter of the protocol for passenger vehicles - Mode, PID, and data bytes. There are 10 Modes in the standard. Mode 1 applies to real-time data such as vehicle speed and RPM, while the 9 other modes are for static data and diagnosis. Within each mode, PIDs exist to represent the various pieces of information. For example, the PID "0D" in Mode 1 represents the real-time vehicle speed. To get this information, the connected device will send something like:

7DF 02 01 0D 55 55 55 55 55

7DF represents a request message, 02 represents the length in bytes of the data, 01 is the mode, and 0d is the requested PID. The 02 tells the vehicle to stop reading data after the second bit, so the rest is filler.

The response, then, could be:

7E8 03 41 0D 32 aa aa aa aa

7E8 represents a response message, 03 is the length in bytes, etc. 32 represents the speed in HEX, which is base-16. Converted to base-10, the speed of the vehicle is:

(16^1 * 3) + (16^0 * 2) = 50 mph. 

Repeat this request several times a second for several PIDs to simulate the "real-time" vehicle speed, RPM, etc.

This overall process is something we can automate in a microprocessor, such as an Arduino UNO connected to an ELM327 chipset. The microprocessor can package this data in some data structure and display it directly to a GUI on an Arduino Shield. Alternatively, the Arduino can pass the decoded and neatly formatted data wirelessly to another device. In this case, we would use an Android phone. The advantage here is that the Android phone has the documentation support for and easy access to an advanced suite of sensors and online information. Here, we would build the GUI in the form of an application that would integrate the Arduino's telemetry stream, the phone's GPS/Acceletrometer/Gyroscope, and anything else desired. To actually display the information to the user, the phone will need to attach to a mount located somewhere in front of the dashboard. A contraption that holds special glass will create a reflect the phone screen and focus the text at infinity. This ensures the user focuses their vision down the road without any of the graphics causing a distraction.

## FOR VEHICLE TELEMETRY
VEHICLE --> 
OBD-II PORT --> 
ELM327 CHIPSET --> 
COM PORT SERIAL STREAM --> 
SocketCAN OR pyOBD PARSER --> 
BUFFER --> 
ANDROID LAYER APPLICATION

## FOR HIGHER LEVEL DATA
GOOGLE MAPS API -->
COMPASS/ELEVATION/ACCELEROMETER/GYRO TELEMETRY -->
ANDROID LAYER APPLICATION
