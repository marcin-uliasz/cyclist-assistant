
<img src="./media/image-05.png" width="32%"/>
<img src="./media/image-10.png" width="32%"/> 
<img src="./media/image-11.png" width="32%"/>

# 1. System description

The system developed by the author is the system for cyclists, which allow them to monitor the cyclist parameters (speed, distance, cadence, calories usage, trip time).
A cyclist can view those parameters on his smartphone. In order to view such data, cyclist needs to install an Android application. The name of the application is Cyclist Assistant. As the name suggest, application provides some assistance during cycling. The assistance in this context are messages displayed on smartphone’s screen, suggesting the cyclist gear usage, depending on various factors. Those factors are a bicycle type, a current gear, a current slope of the terrain. The application offers cyclists an additional functionality. Cyclist can create multiple accounts, add multiple bicycles and view statistics of a current and all trips.

# 2. Software implementation

The whole system for cyclist consist of the Android application, running on a smartphone and server application running on a computer platform, placed on a bicycle. Android application is written in Java language. Server application is written in JavaScript, which runs with NodeJS – cross-platform runtime environment for server-side and networking application”.

The name of a computer platform is RaspberryPi – system on chip platform.

Android application communicates with server application via Bluetooth RFCOMM protocol. Messages are serialized JSON objects. JSON objects sent from smartphone application contain method name to be invoked on a RaspberryPi and additional parameters. The structure of JSON object complies with JSON-RPC specification. Server application binds with SQLite3 database and uses third party libraries for serial port and GPIO access.

## 2.1 Architectural patterns

In different cases the smartphone and the RaspberryPi board can be seen as a client and server or as two peers.

### 2.1.1 Client – Server

<img src="./media/image-01.png" width="90%"/>

### 2.1.2 Peer to Peer

<img src="./media/image-02.png" width="90%"/>

## 2.2 Application features

After launching the application, it checks, does the smartphone, on which the application is running, have Bluetooth module. If module is not available, dialog informing user, that application cannot function properly, will appear. Otherwise, if Bluetooth is available, application enables it in background. After enabling Bluetooth, process of establishing connection to the server on RaspberryPi board starts:

<img src="./media/image-03.png" width="32%"/>

Problem with server (RaspberryPi is disabled or RaspberryPi is enabled, but server did not start yet) will cause an error and displaying message, as shown below:

<img src="./media/image-04.png" width="32%"/>

If connection with server was established, proper message informs user about that fact:

<img src="./media/image-05.png" width="32%"/>

### 2.2.1 New user

After application established connection with the server, user can login or create new account (add new user).
New user form is divided into two sections: a profile and a bicycle. 

Profile section contains fields for entering username, password and user’s weight. 
Correct weight is needed for calculating calories consumption, during bicycle rides.

Bicycle section allows to enter all relevant information about user’s bicycle. 
After logging in, user can add more bicycles, so specifying name of bicycles, by populating bicycle name field let user distinguish multiple bicycles.

Selecting bicycle type is needed for algorithm generating gear usage suggestions and speed warnings.

Entering bicycle weight is needed for correct calculation of calories consumed by user.

After selecting rim type and tire size, wheel perimeter field is populated with relevant value, so user does not have to measure his bicycle’s wheel perimeter.

Correct wheel perimeter is needed for speed calculation.

Wheel perimeter field is editable, so user can change the value, even after the value was inserted into that field automatically.

All those fields are visible below:

<img src="./media/image-06.png" width="32%"/>

### 2.2.2 Login

On the login form, user enters his username, password and selects one bicycle from the list of his all bicycles.

Username has to be entered in a first place, so that bicycle list can be generated in the background:

<img src="./media/image-07.png" width="32%"/>

### 2.2.3 Dashboard

Dashboard is the graphical interface, displaying multiple parameters during bicycle trip (ride). All those parameters relates to *current* bicycle trip and are present in picture below:

<img src="./media/image-08.png" width="32%"/>

After pressing circle button with horizontal bars, we can see the menu dialog:

<img src="./media/image-09.png" width="32%"/>  

**Speed**

When a magnet attached to a spoke passes speed sensor, placed on bicycle it is understood as one bicycle wheel’s rotation.
Speed unit is km/h.

Speed is calculated on RaspberryPi:

```javascript
SpeedSensor.prototype.calculateSpeed = function()  
{ 
    return (this.perimeter / this.getTickTime()) \ 3.6;  
};
```

**Cadence**

When a magnet attached to crank’s arm passes cadence sensor, placed on bicycle it is understood as one bicycle crank’s revolution. Cadence is number of such revolutions per minute.
Cadence unit is rpm.

Cadence is calculated on RaspberryPi:

```javascript
CadenceSensor.prototype.calculateCadence = function()
{
    return 60000 / this.getTickTime();
};
```

**Distance**

Distance unit is meter.

Distance is calculated on RaspberryPi:

```javascript
//once magnet pass sensor this line of code is executed
classScope.distance += classScope.perimeter;

//distance is available
SpeedSensor.prototype.getDistance = function()
{
    return this.distance / 1000;
};
```

**Gear inches**

Gear inches is dimensionless. It is ratio.

Gear inches are calculated on RaspberryPi:

```javascript
function calculateGearInches(speed, cadence)
{
    var gearInches = (speed / 656.167979) / (cadence / Math.PI);

    if (gearInches === Infinity || isNaN(gearInches))
    {
        gearInches = 0;
    }

    return gearInches;
}
```

**Slope**

Slope is calculated on the smartphone, using accelerometer and magnetometer sensor.
Slope unit is percent \[%\].

**Calories**

Calories are calculated using special formula.
Calorie unit is Kcal.
Calories are calculated on the smartphone, because they depend on slope value, which can be calculated easiest on the smartphone.

**Gear icon and corresponding message**

Each bicycle type has set in code recommended gear inches ranges for different type of slopes. These information and current slope and gear inches are compared every 500ms and result of the comparison are three possible messages:
- Use lower gear
- Use higher gear
- You are using optimal gear

**Cadence icon and corresponding message**

Experience of many cyclists indicates that optimal cadence range is 70 – 100. (http://inl.org/cycling/advice/cadence/)

Function responsible for cadence messages, returns three possible messages:
- If current cadence is less than 70: Cadence is to low
- If current cadence is between 70 and 100: Cadence is optimal
- If current cadence is greater than 100: Cadence is to high

**Calories icon**

When icon is red, calories are actively consumed (cyclist is pedalling).
When icon is green calories are not actively consumed (cyclist is not pedalling, but bicycle is going).

**Speed warning icon and corresponding message**

Each type of bicycle has speed limit specified. 
If current bicycle speed is greater than the limit, then speed warning message appears and speed icon becomes red.

<img src="./media/image-10.png" width="32%"/>

### 2.2.4 Statistics

Statistics are divided into two sections: current bicycle trip and all bicycles trip. 
Current bicycle trip section contains trip time, average speed, max speed and max cadence.

<img src="./media/image-11.png" width="32%"/>

All bicycle trips sections contains all above mentioned parameters, as well as total distance.

### 2.2.5 Settings

Settings are divided into four sections: currently selected bicycle’s, bicycles’, cyclist’s profile and slope sensor’s section. All settings are discussed in details in sections A – H.

<img src="./media/image-12.png" width="32%"/>

**A) Bicycle is loaded with bags**

Toggle allowing user to select whether his bicycle is loaded with bags or not. This setting reflects displaying gear usage suggestions and cyclist’s calories consumption.

<img src="./media/image-13.png" width="26%"/>

When bicycle is loaded with bag it is heavier. This fact causes that calories consumption is higher and “use lower gear” suggestion will appear earlier on less steep terrain.

**B) Change bicycle**

Field allowing user to change his current bicycle.  
(Assuming system is enabled and user placed RaspberryPi board and smartphone on another bicycle).  
After touching the field, confirmation dialog appears:  

<img src="./media/image-14.png" width="26%"/>  

When user will touch **No** dialog will be closed and no further action will be taken.  
When user will touch **Yes**, dialog with bicycles list of current user will appear, as it is showed below:  

<img src="./media/image-15.png" width="26%"/>  

After selecting certain bicycle dashboard and statistics of current trip will be reset. 
Server on RaspberryPi will start measuring parameters taking into account details of newly selected bicycle  
(wheel perimeter, bicycle type and weight).  

**C) Manage bicycles**

Manage bicycles form allows user to add new bicycles, edit and remove existing ones:

<img src="./media/image-16.png" width="32%"/>  

User can open this form, by touching **Manage** button:  

<img src="./media/image-17.png" width="26%"/>  

Bicycle marked as green is currently selected:  

<img src="./media/image-18.png" width="32%"/>  

<img src="./media/image-19.png" width="32%"/>

**D) Show tabs names**  

<img src="./media/image-20.png" width="26%"/>  

User can decide, whether tabs names should be visible or not.  
When they are visible, then currently visible tab is highlighted. Tabs names are also touchable.  
Hiding tabs, user can gain some space for the graphical interface.  

**E) Edit profile**  

<img src="./media/image-21.png" width="26%"/>

User has got an option for editing his profile. In order to open edit form, he has to touch **Edit** button.  

<img src="./media/image-22.png" width="32%"/>  

<img src="./media/image-23.png" width="32%"/>

**F) Remove profile**  

<img src="./media/image-24.png" width="26%"/>

User has got an option to remove his profile.  

<img src="./media/image-25.png" width="26%"/>  

After touching remove button, profile, statistics and all bicycles of current user will be removed.  
After successful removal, the screen shown below is visible:  

<img src="./media/image-26.png" width="32%"/>  

**G) Calibrate slope sensor**  

<img src="./media/image-27.png" width="26%"/>  

Slope sensor is a simplification for the accelerometer and magnetometer sensors, located in a smartphone, which are used to calculate the slope of a terrain. Slope is based on the position of a smartphone. When user place the smartphone on his bicycle’s handlebar, accelerometer and magnetometer sensors can be used to calculate a slope of a terrain.

User, in order to see the screen of the smartphone, during cycling, will place it at certain angle. Such angle will false the real slope of a terrain. Calibrate function is available in order to provide the correct value of a slope, even if the smartphone is placed on a handlebar at an angle. Before a user can calibrate the slope sensor, he needs to make sure that the smartphone is placed at the right, comfortable angle and the angle won’t be changed during cycling. Once the smartphone is placed correctly, user needs to find perfectly flat area (for example a floor in a house), then he can calibrate the sensor. Calibration will set slope to 0%, previous value will be remembered and subtracted from current value, every time it will change. In other words, percentage offset will be applied.  

**H) Reset slope sensor**  

<img src="./media/image-28.png" width="26%"/>

After slope sensor calibration user can always reset slope sensor.  
Resetting will remove a percentage offset and slope sensor will output the real values.  

## 3. Hardware design

### 3.1 Circuit diagrams

Circuit diagram shown below demonstrates the construction of the device capable of doing computation, actuation and sensing. Embedding all electronic components into a bicycle, what are present on the diagram, would make a bicycle the real Thing, according to IoT definition, but for needs of project the author placed them in a plastic case, which is situated on a bicycle’s frame.

<img src="./media/image-29.png" width="66%"/>

Next 3 diagrams demonstrates how the device functions in different cases.  

The device batteries are charged by electrical current source – a dynamo. The device is disabled. Bicycle is going.  

<img src="./media/image-30.png" width="66%"/>  

The device is connected with dynamo and is enabled. Batteries are not charged. Bicycle is going.  

<img src="./media/image-31.png" width="66%"/>  

The device is enabled and powered by the batteries. Bicycle is not going.  

<img src="./media/image-32.png" width="66%"/>  

Batteries play important role in the device, because they allow to work the device, regardless of bicycle state (going / not going).

The project required connecting two reed switches into the RaspberryPi board.
The reed switch is an electrical switch operated by an applied magnetic field.  

<img src="./media/image-33.jpeg" width="32%"/>  

First reed switch is required to monitor speed, distance and time of a bicycle trip.
Second reed switch is required to monitor cadence.
Combination of two reed switches allows also for calculation of gears ratio: gear inches.  

Below is a circuit diagram demonstrating, how reed swithces are connected with the RaspberryPi GPIO:  

<img src="./media/image-34.jpg" width="40%"/>

### 3.2 GPIO

<img src="./media/image-35.jpeg" width="32%"/>  

GPIO is the interface for communication between components of a computer system, such as a microprocessor or various peripherals. Outputs (pins) of such a device can serve both as inputs and outputs and are usually configurable. GPIO pins are often grouped in ports.

<img src="./media/image-36.jpeg" width="32%"/>  

## 4. Hardware implementation

### 4.1 Parts 

List of hardware and parts used in the project:  

<img src="./media/image-37.png" width="100%"/>

### 4.2 RaspberryPi board (computer platform)

Project required to place the RaspberryPi board in a plastic case, together with the batteries and the voltage regulator. The author had to modify the original cables of reed switches, so that they could be connected to the GPIO pins on the RaspberryPi board. Bluetooth dongle was modified as well – it was soldered with external antenna.  

Plastic case was attached to the bicycle.  

Reed switches and magnets was placed on the one spoke of front bicycle wheel and on the arm of crank. Cables of reed switches were connected to the plastic case via Molex sockets.  

<img src="./media/image-38.png" width="65%"/>

### 4.3 Smartphone

Smartphone model used in a project was Samsung Galaxy GT-I9100.  
The smartphone was placed on a bicycle’s handlebar, in a special holder.

## Appendix  

Third party libraries, resources and tools used in the project:  

**LIBRARIES**

**Android libraries:** 

> **PagerSlidingTabStrip**
“Interactive paging indicator widget, compatible with the ViewPager from the Android Support Library.”  
Version: 1.0.9  
Page: https://github.com/jpardogo/PagerSlidingTabStrip  

> **Android-Bootstrap**
Bootstrap based style for Android  
Page: https://github.com/Bearded-Hen/Android-Bootstrap  

> **AndroidSVG**  
AndroidSVG is a SVG parser and renderer for Android  
Version: 1.2.1  
Page: https://code.google.com/p/androidsvg/  

> **ToggleButton**  
ToggleButton Widget For Android Developers  
Page: https://android-arsenal.com/details/1/1158  

> **EventBus**  
EventBus is publish/subscribe event bus optimized for Android. Simplifies the communication between components  
Version: 2.4.0  
Page: https://github.com/greenrobot/EventBus  

> **ButterKnife**  
Field and method binding for Android views which uses annotation processing to generate boilerplate code for you  
Version: 6.1.0  
Page: https://github.com/JakeWharton/butterknife  

**JavaScript libraries**

> **Justgage**  
JustGage is a handy JavaScript plugin for generating and animating nice & clean dashboard gauges. It is based on Raphaël library for vector drawing.  
Version: February 16, 2014  
Home page: http://justgage.com/\#setup  

**NodeJS modules:**  

> **onoff**  
GPIO access and interrupt detection with io.js or Node.js on Linux boards like the BeagleBone, BeagleBone Black, Raspberry Pi, or Raspberry Pi 2.  
Version: 1.0.2  
Page: https://github.com/fivdi/onoff  

> **serialport**  
It provides a very simple interface to the low level serial port code necessary to program Arduino chipsets, X10 wireless communications, or even the rising Z-Wave and Zigbee standards  
Version: 1.6.1  
Page: https://github.com/voodootikigod/node-serialport  

> **sqlite3**  
Asynchronous, non-blocking SQLite3 bindings for Node.js.  
Version: 3.0.5  
Page: https://github.com/mapbox/node-sqlite3  

**RESOURCES**

> **SVG icons:**  
Page: http://www.flaticon.com/free-icon/mountain-cycling\_37961  
Icon made by Freepik from www.flaticon.com

> **Fonts:**  
Font Awesome gives you scalable vector icons that can instantly be customized: size, color, drop shadow, and anything that can be done with the power of CSS  
Version: 4.3.0  
Home page: http://fortawesome.github.io/Font-Awesome/
