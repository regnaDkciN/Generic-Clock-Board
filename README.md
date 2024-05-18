# Generic Clock Board

An ESP32 based PC board that is specifically designed to support electronic clocks.  Supports the following:
- Adafruit Huzzah32 - ESP32 Feather board for control.
- Optional Adafruit DS3231 Real Time Clock module.
- Optional 5 volt ULN2003 stepper motor driver with optional phase display LEDs (mainly intended for use with a 28BYJ-48 stepper motor).
- Optional home input for homing the stepper motor, if used.  Can be used for other purpose if home is not needed.
- RGB LED for status display.
- Pushbutton input for general clock use.
- Optional 2 auxiliary I/O which can be individually used as inputs or outputs as needed.
- All components are through hole for easy construction.
- Supporting Arduino IDE board driver written in C++.

The motivation for this board was gzumwalt's excellent Geneva Clock which was originally published on printables and cults3d:
[ https://www.printables.com/model/717033-geneva-clock](https://www.printables.com/model/717033-geneva-clock) and
 [https://cults3d.com/en/3d-model/home/geneva-clock](https://cults3d.com/en/3d-model/home/geneva-clock)

---
## Parts List
- 1 Generic Clock Board (files attached).
- 1 [Adafruit Huzzah32-ESP32](https://www.adafruit.com/product/3405).
- 1 **Optional** [Adafruit DS3231 Real Time Clock](https://www.adafruit.com/product/3013).  Used to provide battery backed time in case power is lost or an internet connection cannot be made.
- 1 **Optional**[Right Angle Tactile Pushbutton Switch](https://www.amazon.com/dp/B008DS16PO?psc=1&ref=ppx_yo2ov_dt_b_product_details).  Pushbutton for general clock use.
- 1 **Optional** [ULN2003 High Voltage Driver](https://www.amazon.com/ALLECIN-ULN2003-ULN2003APG-High-Voltage-High-Current/dp/B0CBM23ZJ3/ref=sr_1_5?sr=8-5).  Used to drive a 5 volt stepper motor.
- 4 **Optional** LED (color as desired).  Used to display stepper motor phase selection, which might be useful for debugging stepper.  Otherwise, not really needed.
- 4 **Optional** 10K resistors.  Used only if the optional LEDs are included.
- 1 **Optional** [5 Pin JST XH Male Connector](https://www.amazon.com/dp/B09JP2D9BH?ref=ppx_yo2ov_dt_b_product_details).  Used to connect to the stepper motor, if used.
- 1 **Optional** [5mm RGB LED](https://www.amazon.com/EDGELEC-Tri-Color-Multicolor-Diffused-Resistors/dp/B077XGF3YR/ref=sr_1_5?sr=8-5).  To display general status if desired.
- 3 **Optional** 330 ohm resistors.  Supports RGB LED, if used.
- 1 **Optional** [2-pin JST male connector](https://www.amazon.com/daier-Micro-2-Pin-Connector-Female/dp/B01DUC1O68/ref=sr_1_9?sr=8-9).  Used for home sensor connector.
- 1 **Optional** [4-pin JST male connector](https://www.amazon.com/Sets-2-5-4-Connector-200mm-Female/dp/B01DUC1S14/ref=sr_1_4?sr=8-4).  For the optional auxiliary connector.

---

## Generic Clock Board Library

The included interface library provides convenient use of the basic elements of the board.  Interfaces exist for the stepper, LEDs, home sensor, and pushbutton switch.  The library consists of the following files:
- GenericClockBoard.h - Declares the GenericClockBoard class.
- GenericClockBoard.cpp - Contains the GenericClockBoard class method implementation.
- SerialDebugSetup.h - Selects compile time options for the SerialDebug library which is used to generate serial test and debug output for the library.

The GenericClockBoard library requires the following libraries to be installed in the Arduino IDE:
- [RGBLed Library](https://github.com/wilmouths/RGBLed): This library supports the board's RGB LEDs and provides dimming and color mixing support.
- [SerialDebug Library](https://github.com/JoaoLopesF/SerialDebug ): This library supports serial debugging and status display.

## StepperSpeed_t enum
This enum is used to select the speed profile that will be used for stepper movement via the Step() method.  Selections are:
- *__StepSlow__* - Will move the stepper at slow speed for the full duration of the move.
- *__StepAuto__* - Will make long moves use the fast speed with accel and decel at the beginning and end of the move.  Short moves will be done at slow speed.
- *__StepFast__* - Will move the stepper at fast speed for the full duration of the move.


## Generic Clock Board Library API
### Constructor
Constructs a class instance, initializes board hardware, and initializes instance variables.
#### Constructor Arguments:
 - *__rapidSecondsPerRev__*  - (uint32_t) Specifies the number of seconds it takes the stepper to make one full revolution of its output shaft.  For the 28BYJ-48 stepper motor, a good range is normally between 6 and 10 seconds.
- *__fullStepsPerRev__* - (uint32_t) Specifies the number of FULL steps per revolution of the stepper motor's output shaft.  For the   28BYJ-48 the value is 2048.
- *__stepperPinsReversed__* - (bool) Specifies the whether or not the stepper turns clockwise when a positive step value is commanded.  Set to 'true' if a positive step value causes counterclockwise movement.  Set to 'false' otherwise.
- *__stepperHalfStepping__* - (bool) Specifies whether half stepping is to be used.  If 'true', then half stepping is used, which will cause the number of steps per rev of the stepper to double.  For example, the 28BYJ-48 stepper will take 4096 steps per rev if this value is set to 'true'.  In most cases, use of half stepping is a good choice.
- *__homeNormallyOpen__* - (bool) Specifies the type of sensor used for homing the clock.  Set to 'true' for normally open  (N.O.) sensors.  Set to 'false' for normally closed (N.C.) sensors.

#### Constructor Example
```
// RAPID_SECONDS_PER_REV specifies the number of seconds it takes for the stepper
// motor to complete 1 full revolution at its maximum speed.  A good value for
// the 28BYJ-48 stepper motor is usually in the range of 6 to 10 seconds.
const uint32_t RAPID_SECONDS_PER_REV = 8;

// 28BYJ-48 has 2048 full steps per full rev of the output shaft (4096 half steps).
const uint32_t FULL_STEPS_PER_REV = 2048;

// This stepper needs to have the phases reversed.  Set to 'false' if stepper runs
// backwards.
const bool REVERSE_STEPPER = true;

// USE_HALF_STEPPING selects the use of half stepping of the stepper motor.
// Half stepping is recommended, and is specified by setting this constant to
// ture.  If full stepping is desired, set it to false.
const bool USE_HALF_STEPPING = true;

// The home sensor is normally open.  Set to false if normally closed.
const bool HOME_SWITCH_NORMALLY_OPEN = true;

GenericClockBoard gClock(RAPID_SECONDS_PER_REV, FULL_STEPS_PER_REV,  
                         REVERSE_STEPPER, USE_HALF_STEPPING, HOME_SWITCH_NORMALLY_OPEN);
```

### Step()
Step the stepper motor a specific number of steps in a specified direction at a specified speed.
#### Step() Arguments
- *__steps__* - (int32_t) Specifies the number of steps and direction that the motor will move.  A positive value will move the motor in the clockwise (CW) direction.  A negative value will move the motor in the counterclockwise (CCW) direction.
- *__speed__* - (StepperSpeed_t) Specifies the speed profile that will be used for the move.  StepSlow selects slow stepper speed.  StepAuto selects automatic stepper speed with accel and decel.  StepFast selects fast stepper speed.
#### Step() Example
```
	// Move the stepper 100 steps CW with the Auto profile.
	gClock.Step(100, StepAuto);
```
---
### IsHome()
Returns 'true' if the home sensor is active, based on the type of sensor (N.O. or N.C.) in use.  Returns 'false' otherwise.

#### IsHome() Example
```
	// Move the stepper slowly CW until home is found.
	while (!gClock.IsHome()
	{
		gClock.Step(1, StepSlow);
	}
```
---

### IsButtonPressed()
Returns 'true' if the board's pushbutton is active.  Returns 'false' otherwise.

#### IsButtonPressed() Example
```
	// Set the RGB LED to magenta if the button is pressed.
	if (gClock.IsButtonPressed())
	{
		gClock.RgbLed.brightness(RGBLed::MAGENTA, 2);
	}
```
---
### GenericClockBoard Public Data

#### RGB LED
The RGB LED is normally controlled via an instance of the RGBLed class that is created in the GenericClockBoard constructor.  The RGBLed instance is defined as:
```
    // RGBLed instance (public for easy access).
    static RGBLed RgbLed;
```
See the [RGBLed library documentation](https://github.com/wilmouths/RGBLed) for details on use.  An example of RgbLed is:
```
	// Set the RGB LED color to red with a brightness of 2.
	gClock.RgbLed.brightness(RGBLed::RED, 2);
```
The I/O pin assignments for the RGB LED are exposed as public constants, and may be used to control the LED if needed, but normally the RgbLed member should be used.  The LED pin assignments are defined as:
```
    static const uint8_t LED_RED_PIN   = 13;  // Red LED output pin assignment.
    static const uint8_t LED_GREEN_PIN = 12;  // Green LED output pin assignment.
    static const uint8_t LED_BLUE_PIN  = 27;  // Blue LED output pin assignment.
```
---

#### Auxiliary I/O Pin Assignments
The auxiliary I/O pin assignments are exposed for public use as follows:
```
    static const uint8_t AUX_1_PIN      = 15;  // Aux1 I/O pin assignment.
    static const uint8_t AUX_2_PIN      = 33;  // Aux2 I/O pin assignment.
```  
 These pins are not initialized or used in any way by the GenericClockBoard class, and can be used as desired.
 
---

#### Stepper Motor Direction Constants

The following constants may be used with the Step() method to assist in commanding the direction of stepper movement:
```
    static const int32_t STEP_CW        = 1;   // Clockwise specifier.
    static const int32_t STEP_CCW       = -1;  // Counterclockwise specifier.

```

An example of the use of these constants is:
```
	// Move 5 steps in the CCW direction.
	gClock.Step(STEP_CCW * 5, StepAuto);
```
---

### Private Constants
Several private constants may be of interest to users who do not want to use the provided GenericClockBoard code.  These constants identify the remainder of the I/O pins used by the board.
```
    // Stepper related constants.
    static const uint8_t PHASE_1_PIN = 19;
    static const uint8_t PHASE_2_PIN = 16;
    static const uint8_t PHASE_3_PIN = 17;
    static const uint8_t PHASE_4_PIN = 21;

    // I/O pin assignments.
    static const uint8_t HOME_PIN       = 32;   // Home input pin assignment.
    static const uint8_t PUSHBUTTON_PIN = 26;   // Pushbutton input pin assignment.

```
---

## General Notes
Interface to the ESP32 WiFi is left to the user.  The prototype code uses the 
[WiFiTimeManager library](https://github.com/regnaDkciN/WiFiTimeManager) for WiFi and DST management.  Similarly, interface to the DS3231 RTC is left to the user.  The prototype code uses the [DS323x_Generic Library](https://github.com/khoih-prog/DS323x_Generic) for RTC interface.

An example of using the full features of this board, including WiFi, DST management, RTC, RGB LEDs, and all other hardware can be found at [my GenevaClock git site](https://github.com/regnaDkciN/GenevaClock).


Next revision (if any) may include:
- Add an input for higher voltage ULN3003 feed for more capable stepper capability.
- Tie Home input and Selector Switch input to +5 volts instead of ground so that ESP32 internal pulldowns can be used to eliminate inversion in code.