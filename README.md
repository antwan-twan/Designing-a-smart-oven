# Designing a (part of) a Smart Oven with ESP8266
Today, I'm designing a piece of a concept I came up with for a project for school. 

Its a smart oven connected via IoT. 

I'll be making a tiny part of the interface using a Servo-motor and an ESP8266 board (also known as NodeMCU 1.0 (ESP-12E Module)). 

## 1. Setup
For this experiment, you'll need a:
- Computer with Arduino IDE installed
- An Adafruit IO account
- An ESP8266 board
- A Servo-motor

and maybe some cables to connect your Servo to the board. Although this depends on what kind a connector you have on your Servo.

---
### Adafruit IO Setup
First of all create an account on [Adafruit IO](io.adafruit.com).
Afterwards, go to your Account settings and retrieve your key.

<img src="img\key.png" width="375px" alt="Adafruit IO Key">

#### Create your feed
In Adafruit IO, create a new feed with the name Servo. 

<img src="img\feed.png" width="375px" alt="feed">

#### Create a new dashboard
Also create a new dashboard.
You can name this whatever you want, but make sure to remember it.

<img src="img\newdashboard.png" width="375px" alt="dashboard">

In this dashboard, create a new block and choose slider block.
It'll ask you what kind of feed you want to connect to your block. Choose the feed you have created in the previous tip.
Set the minimum value to zero and the maximum value to 180. You can leave the rest empty.

<img src="img\blockset.png" width="375px" alt="block settings">


## 2. Wiring
Now it's time to connect your wires to your servo motor and your Arduino Board.

You will need to connect the following pins to the servo using the three wires:
- Connect GND to the brown or black servo wire
- Connect 3V3 to the red servo wire
- Connect Pin D2 to the yellow servo wire

I've used an extension cable but it should work the same. Follow the origninal colors of the servo cables. This maybe a bit difficult, but it will help in the end. 

<img src="img\wiring.png" width="375px" alt="servo wiring">

## 3. Arduino Code
This one is quite simple, you Just need to connectthe Arduino Library to your Arduino IDE. 
In the library manager, search for Adafruit IO Arduino by Adafruit and install the latest version.

<img src="img\library.png" width="375px" alt="library manager">

If all goes well, open in File > Examples > Adafruit IO Arduino > adafruitio_16_servo.

### Network Config
The example files are already stocked with information. All of the details you need to fill in are provided.

Go to the *config.h* file and fill in your Adafruit IO username and key as well as your WiFi-network name and password. 

<img src="img\config.png" width="375px" alt="config.h">

> :warning: In my case I'm using an iPhone hotspot, which means I have to turn on the 'Maximise Compatibility' function in order for the board to correctly connect to the internet.
>
> :warning: Otherwhise, make sure that your Android Hotspot or WiFi-router is capable of 2,4gHz networking. This is the **only way** to connect. 

> :warning: The guide I'm following also states tha you could uncomment some lines of code for Ethernet or FONA config, but skip those both.

### Changes in the code
You don't need to change much to the code. 

On line 32: change the SERVO_PIN to 'D2'.

On line 38: change the feed name to the feed you've created. In my case, I had to change it to 'Servo' with a capital 'S'. 

<img src="img\code.png" width="375px" alt="code changes">

Upload your code, but make sure to compile first, to work out any errors. 

> :pencil: Somehow the angle of the Servo in combination with the Adafruit IO Dashboard was inverted. Meaning that when I tell the arduino to write an angle of 180, it goes all the way to the left, which is zero.
>
> After doing some research, I've managed to find a solution for this problem. It's quite an easy fix. 
> Convert the line "int angle = data->toInt();" to 'int angle = 180 - data->toInt();". This fixes the output by a simple equation. 

### Adding a LEDstrip
After the servo was done, I tried adding a LEDstrip to work together with the data. 

To do this, follow these steps:
1. Add the following code somewhere at the beginning of the file, but before any void setup/loops.
```
    #include <Adafruit_NeoPixel.h>

    #define PIN D5

    #define NUMPIXELS 10
    int i = 0;
    Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);
```

2. In 'void handleMessage' add the following code:
```
     if(angle < 90){
    pixels.setPixelColor(i, pixels.Color(0, 150, 0));
    pixels.show();
    }else if(angle > 90){
    pixels.setPixelColor(i, pixels.Color(150, 0, 0));
    pixels.show();
    }
```

> :warning: Somehow, only one LED of my LEDstrip was turning on. I don't know what causes this problem to occur. However, I've encountered this problem before while working at a project for another course. It's related to the power distribution in the Arduino. Both the Servo-motor and the LEDstrip require quite some power and somehow the board can't handle this.






## 4. Results

[Video Results](https://vimeo.com/764351321).



<details><summary>Source Code</summary>

```
    // Adafruit IO Servo Example
    // Tutorial Link: https://learn.adafruit.com/adafruit-io-basics-servo
    //
    // Adafruit invests time and resources providing this open source code.
    // Please support Adafruit and open source hardware by purchasing
    // products from Adafruit!
    //
    // Written by Todd Treece for Adafruit Industries
    // Copyright (c) 2016-2017 Adafruit Industries
    // Licensed under the MIT license.
    //
    // All text above must be included in any redistribution.

    /************************** Configuration ***********************************/

    // edit the config.h tab and enter your Adafruit IO credentials
    // and any additional configuration needed for WiFi, cellular,
    // or ethernet clients.
    #include "config.h"

    /************************ Example Starts Here *******************************/

    #if defined(ARDUINO_ARCH_ESP32)
    // ESP32Servo Library (https://github.com/madhephaestus/ESP32Servo)
    // installation: library manager -> search -> "ESP32Servo"
    #include <ESP32Servo.h>
    #else
    #include <Servo.h>
    #endif

    // pin used to control the servo
    #define SERVO_PIN D2

    // create an instance of the servo class
    Servo servo;

    // set up the 'servo' feed
    AdafruitIO_Feed *servo_feed = io.feed("Servo");

    #include <Adafruit_NeoPixel.h>

    #define PIN D5
    #define NUMPIXELS 10
    int i = 0;
    Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);






    void setup() {
    pixels.begin();

    // start the serial connection
    Serial.begin(115200);

    // wait for serial monitor to open
    while(! Serial);

    // tell the servo class which pin we are using
    servo.attach(SERVO_PIN);

    // connect to io.adafruit.com
    Serial.print("Connecting to Adafruit IO");
    io.connect();

    // set up a message handler for the 'servo' feed.
    // the handleMessage function (defined below)
    // will be called whenever a message is
    // received from adafruit io.
    servo_feed->onMessage(handleMessage);

    // wait for a connection
    while(io.status() < AIO_CONNECTED) {
        Serial.print(".");
        delay(500);
    }

    // we are connected
    Serial.println();
    Serial.println(io.statusText());
    servo_feed->get();

    }

    void loop() {

    // io.run(); is required for all sketches.
    // it should always be present at the top of your loop
    // function. it keeps the client connected to
    // io.adafruit.com, and processes any incoming data.
    io.run();

    }

    // this function is called whenever a 'servo' message
    // is received from Adafruit IO. it was attached to
    // the servo feed in the setup() function above.
    void handleMessage(AdafruitIO_Data *data) {

    // convert the data to integer
    int angle = 180 - data->toInt();

    // make sure we don't exceed the limit
    // of the servo. the range is from 0
    // to 180.
    if(angle < 0)
        angle = 0;
    else if(angle > 180)
        angle = 180;

    servo.write(angle);

    if(angle < 90){
        pixels.setPixelColor(i, pixels.Color(0, 150, 0));
        pixels.show();
        }else if(angle > 90){
        pixels.setPixelColor(i, pixels.Color(150, 0, 0));
        pixels.show();
        }

    }
```

</details>



# Sources
- https://learn.adafruit.com/adafruit-io-basics-servo/overview
- https://forum.arduino.cc/t/servo-reversing-with-code/878196/6