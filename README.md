# vicHUD
Android/Arduino based Vehicle Heads-Up-Display for vehicles after 2002 using OBD-II ports and CAN protocol

This is a base README file for the project. Overall plan for the vicHUD:

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
