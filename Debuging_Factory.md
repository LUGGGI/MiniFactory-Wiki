# Debugging Manual for Factory and PLC

## Abbreviations

| Abbreviation | Meaning                                                |
| ----------- | ------------------------------------------------------ |
| SENS        | Light barrier (have an LED and an Light sensor)        |
| REF_SWITCH  | Reference switch, to find a start position for an axis |
| CB          | Conveyer belt                                           |
| GR          | 3D-Robot with claw attachment                          |
| VG          | 3D-Robot with vacuum gripper attachment                 |
| MPS         | Multi Purpose Station                                   |
| PM          | Punching Machine                                       |
| SL          | Sorting Line                                           |
| INDX        | Index line with drill and mill                         |
| WH          | Warehouse                                              |

## Debugging Tools

### Status-LEDs on the PLC

3 LEDs can be found on each controller (the one with the LAN-Port) and 2 on each expansion Modul

#### LEDs on Controller

| LED   | Color          | Meaning                                                                               |
| ----- | -------------- | ------------------------------------------------------------------------------------- |
| Power | green          | RevPi Core running                                                                    |
|       | red            | The piControl driver resets, the modules connected to the PiBridge are reinitialized. |
|       | off            | No Power                                                                              |
| A1    | green blinking | Control Software is running                                                           |
|       | red blinking   | Error/Problem in Factory                                                              |
|       | static/off     | Control Software**not** running                                                 |
| A2    | green blinking | A Production-Line is running                                                          |
|       | red blinking   | Error/Problem in a Production-Line                                                    |
|       | static/off     | No Production-Line is running                                                         |

#### LEDs on expansion modules

| LED   | Signal       | Meaning                                                                                                     |
| ----- | ------------ | ----------------------------------------------------------------------------------------------------------- |
| Power | on, green    | RevPi Core is connected to the power supply.                                                                |
|       | flashes, red | Connection to RevPi Core is not yet set up (initialization phase).                                          |
|       | on, red      | Connection to RevPi Core is interrupted. Check if PiBridge is plugged properly.                             |
| OUT   | off          | Connection to RevPi Core is not yet set up (initialization phase).                                          |
|       | on, green    | Outputs are ready for use.                                                                                  |
|       | flashes, red | Outputs are defective. Check the connection to the power supply.                                            |
|       | on, red      | No or low power supply. Check if your device is supplied by the permitted voltage range (10.7 V – 28.8 V). |
| IN    | on, green    | Inputs are ready for use.                                                                                   |
|       | off          | Connection to RevPi Core is not yet set up (initialization phase).                                          |
|       | on, red      | No power supply to the inputs on the plug X2. Check the connection to the power supply.                     |

### RevPiCommander

Download at: [https://revpimodio.org/en/sources/revpipycommander/]()

Abilities:

* Start/Stop/Restart Control-Software
* Display general Log
* Configure PLC
* Read Inputs
* Set Outputs

#### PLC watch mode (read inputs, set outputs)

When a fault is detected by the Control-Software it is helpful to check the displayed sensor/ corresponding actuator.

It shows live which Sensor is detecting something and which motor is running.

**Stop the Control-Software bevor using it**

### PiTest

Simple Commandlinetool on the PLC. Same functionality as PLC watch mode on the RevPiCommander

Tutorial: [https://revolutionpi.com/en/tutorials/software/pitest-verwenden]()

## Logging Systems

### MQTT-Log

All important events are sent via MQTT

### General-Log

Logs the important events.

* Not persistent over reboot
* Can only be read with RevPiCommander

## Debugging Errors and Problems

If the factory stops or halts you can look at the errors in the mqtt_signals or the log.

How to find the problem sensor or actuator

* The first part of a sensor/actuator name is the machine it is on (`CB1_SENS_END `, the machine is `CB1`)
* Most of the time the error messages includes a sensor even if the sensor isn't the part that has a fault
  * see the examples below for details

### Error examples and the corresponding reasons

* `'PROBLEM': 'SensorTimeoutError: CB1_SENS_END no detection in time'`
  * Product didn't reach the Light Barrier bevor a give timeout is reached
  * Reasons can be:
    * Product ist stuck somewhere or was dropped
    * Problem with the CB-Motor
    * Problem with the Sensor
* `'PROBLEM': 'GetProductError: GR2 :Product still at Sensor, grip failed'`
  * Sensor at gripping position still detect something even after gripping and moving away
  * Reasons can be:
    * Robot position is of, so it tries to grip at the wrong location
    * Problem with Claw-Motor
* `'PROBLEM': 'SensorTimeoutError: GR2_ROTATION_ENCODER :Value 1400 not reached in time'`
  * The encoder for the rotation axis didn't reach the value in time
  * Reasons can be:
    * Problem with Motor
    * Robot stuck / collided with something
    * Problem with encoder (sensor)
* `'PROBLEM': 'SensorTimeoutError: GR2_REF_SW_ROTATION_ENCODER no detection in time`'
  * While running to to reference switch a timeout occurred
  * Reasons can be:
    * Problem with Motor
    * Robot stuck / collided with something
    * Problem with reference switch (sensor)
* `'PROBLEM': 'LookupError: WH: No empty spaces left.' `or `'LookupError: WH: Color RED not found.'`
  * Color or no space found when looking in the `wh_content.json` file.
  * Free up space or update the file with mqtt

#### Reasons for Problems at Motor/Sensor

##### Motor

* Cable lose at the machines pcb
* Cable lose at plc terminal
* Axis/CB etc. is stuck
* A gear is lose
* Motor broken

##### Sensor

* Cable lose at the machines pcb
* Cable lose at plc terminal
* At Light barriers:
  * LED not working
  * LED misaligned
* Sensor broken

### HOW TO fix a Problem

* Find the Sensor/Actuator with the problem
* Look at Error examples and find the reason
  * Use RevPICommander or PiTest to test actuators and sensors without the program running
