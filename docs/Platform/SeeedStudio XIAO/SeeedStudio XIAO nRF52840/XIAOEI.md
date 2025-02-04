---
description: Motion Recognition by XIAO nRF52840
title: Motion Recognition by XIAO nRF52840
image: https://avatars.githubusercontent.com/u/10758833
last_update:
  date: 11/23/2022
  author: Bill
---

# Motion Recognition

In this wiki, we will show you how to utilize the accelerometer on Seeed Studio XIAO nRF52840 Sense combined with Edge Impulse to enable motion recognition. The codes we present here are supported by latest version of Seeed nRF52 Boards.

> When it comes to embedded AI applications, we highly recommend using the "Seeed nrf52 mbed-enabled Boards Library".

<iframe width="560" height="315" src="https://www.youtube.com/embed/hLKKorpDlYw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Materials Required

### Hardware

In this wiki, we need to prepare the following materials:

- [Seeed Studio XIAO nRF52840 Sense](https://www.seeedstudio.com/Seeed-XIAO-BLE-Sense-nRF52840-p-5253.html)
- Li-po battery (702025)
- [Grove - OLED Display 0.66"](https://www.seeedstudio.com/Grove-OLED-Display-0-66-SSD1306-v1-0-p-5096.html)
- Dupont cables or Grove cables
- 3D-printed shell
- Light guide plastic fiber

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/BLEmotion.png" style={{width:800, height:'auto'}}/></div>


**Hardware Set up**

- **Step 1**. Remove the Grove base on Grove - OLED Display 0.66" with a soldering iron
- **Step 2**. Use wire cutters to process the DuPont cables to a length of about 3 cm and expose the inner cables of about 2 mm at both ends
- **Step 3**. Pass the fiber through the small hole in the front and place the end at the LED

- **Step 4**.  Solder Seeed Studio XIAO nRF52840 Sense with other elements according to the diagram below:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition2.png" style={{width:400, height:'auto'}}/></div>

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition3.png" style={{width:400, height:'auto'}}/></div>

:::note:

It will be better if you use hot melt adhesive to reinforce welds.

:::

- **Step 6**. Assemble all components:

  1. Pass the fiber through the small hole in the front of shell
  2. Mount the screen to the fixed location
  3. Sandwich the battery between Seeed Studio XIAO nRF52840 and screen
  4. Handle the wires carefully
  5. Place the end of light guide plastic fiber at the RGB light of Seeed Studio XIAO nRF52840 and cut off the excess
  6. Assemble the case

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition4.png" style={{width:400, height:'auto'}}/></div>

The assemble one:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition6.png" style={{width:400, height:'auto'}}/></div>

### Software

The required libraries are listed below. It is highly recommanded that use the codes here to check whether the hardware is functioning well. If you have problem about installing the library, please refer to [here](https://wiki.seeedstudio.com/How_to_install_Arduino_Library/).

- [Seeed_Arduino_LSM6DS3-master](https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Seeed_Arduino_LSM6DS3-master.zip)
- [U8g2](https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/U8g2.zip)

## Get started

First we are going run some demos to check whether the board and the display screen is functioning well. If yours are fine, you can move on to the next instruction.

### Check the position of light guide plastic fiber

Follow this [tutorial](https://wiki.seeedstudio.com/XIAO_BLE/#getting-started) and run Blink demo.
If you see the red light in the front hole, it means success.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition7.png" style={{width:400, height:'auto'}}/></div>

### Check the circuit connection

Open the Arduino IDE, navigate to Sketch -> Include Library -> Manage Libraries... and Search and Install `U8g2 library` in the Library Manager.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition29.png" style={{width:400, height:'auto'}}/></div>


After the installation, copy the following code run it.

```c
#include <Arduino.h>
#include <U8x8lib.h>
 
// U8X8_SSD1306_128X64_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);
 U8X8_SSD1306_64X48_ER_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE); 
// U8X8_SSD1306_64X32_NONAME_HW_I2C u8x8(/* reset=*/ U8X8_PIN_NONE);
 
// U8X8_SSD1306_128X64_NONAME_SW_I2C u8x8(/* clock=*/ SCL, /* data=*/ SDA, /* reset=*/ U8X8_PIN_NONE);   // OLEDs without Reset of the Display
 
void setup(void) {
  u8x8.begin();
  //u8x8.setFlipMode(2);   // set number from 1 to 3, the screen word will rotary 180
}
 
void loop(void) {
      u8x8.setFont(u8x8_font_amstrad_cpc_extended_r);
      u8x8.drawString(0,0,"idle");
      u8x8.drawString(0,1,"left");
      u8x8.drawString(0,2,"right");
      u8x8.drawString(0,3,"up&down");   
} 
```

After uploading the code and unplugging Seeed Studio XIAO nRF52840, if you see the same result, it means success.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew.png" style={{width:300, height:'auto'}}/></div>


### Check the accelerometer

Upload the code below through Arduino IDE to Seeed Studio XIAO nRF52840:

```c
#include "LSM6DS3.h"
#include "Wire.h"

//Create a instance of class LSM6DS3
LSM6DS3 myIMU(I2C_MODE, 0x6A);    //I2C device address 0x6A


#define CONVERT_G_TO_MS2    9.80665f
#define FREQUENCY_HZ        50
#define INTERVAL_MS         (1000 / (FREQUENCY_HZ + 1))

static unsigned long last_interval_ms = 0;

void setup() {
    // put your setup code here, to run once:
    Serial.begin(115200);
    while (!Serial);
    //Call .begin() to configure the IMUs
    if (myIMU.begin() != 0) {
        Serial.println("Device error");
    } else {
        Serial.println("Device OK!");
    }
}

void loop() {

    if (millis() > last_interval_ms + INTERVAL_MS) {
        last_interval_ms = millis();

        Serial.print(myIMU.readFloatGyroX() * CONVERT_G_TO_MS2,4);
        Serial.print('\t');
        Serial.print(myIMU.readFloatGyroY() * CONVERT_G_TO_MS2,4);
        Serial.print('\t');
        Serial.println(myIMU.readFloatGyroZ() * CONVERT_G_TO_MS2,4);
    }
}
```

Open the serial monitor to check the output:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition9a.png" style={{width:400, height:'auto'}}/></div>

If all is fine, we can move on and connect Seeed Studio XIAO nRF52840 to Edge Impulse.


## Motion Recognition on Seeed Studio XIAO nRF52840 connected with Edge Impulse

The precision of the training model is very important to the final result. If your output training results are as low as less than 65%, we highly recommand you train for more times.

- **Step 1.** Create a new project in [Edge Impulse](https://studio.edgeimpulse.com/)

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition9.png" style={{width:700, height:'auto'}}/></div>

- **Step 2.** Choose "Accelerometer data" and click "Let’s get started!"

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition10.png" style={{width:700, height:'auto'}}/></div>

- **Step 3.** Install [Edge Impulse CLI](https://docs.edgeimpulse.com/docs/cli-installation) in your computer.

- **Step 4.** Run the command in your `terminal` or `cmd` or `powershell` to start it.

```
edge-impulse-data-forwarder
```

- **Step 5.** We need to use the CLI to connect the Seeed Studio XIAO nRF52840 Sense with Edge Impulse. First, login your account and choose your project

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition11.png" style={{width:700, height:'auto'}}/></div>

Name the accelerometer and the device.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition12.png" style={{width:700, height:'auto'}}/></div>

Move back to Edge Impulse "Data acquisition" page, the outcome should be like this if the connection is successful. You can find the Device of "Seeed Studio XIAO nRF52840 Sense" is shown on the right of the page. 

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition13.png" style={{width:800, height:'auto'}}/></div>

- **Step 6.**  Select the sensor as "3 axes". Name your label as `up` and `down`, modify Sample length (ms.) to 20000 and click start sampling.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition13.png" style={{width:800, height:'auto'}}/></div>

- **Step 7.** Swing the Seeed Studio XIAO nRF52840 Sense up and down and keep the motion for 20 seconds. You can find the acquistion is shown up like this:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition14.png" style={{width:800, height:'auto'}}/></div>

- **Step 8.** Split the data by clicking the raw data right top and choose "Split Sample". Click +Add Segment and then click the graph. Repeat it more than 20 time to add segments. Click Split and you will see the the sample data each for 1 second.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition30.png" style={{width:800, height:'auto'}}/></div>

- **Step 9.** Repeat **Step 7.** and **Step 8.** and label data with different name to click different motion data, like `left` and `right`, `clockwise`, `anticlockwise` and so on. The example provided is classifying up and down, left and right, and circle. You can change it as you may want to change here.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition16.png" style={{width:800, height:'auto'}}/></div>

:::note:

In Step 8. the split time is 1 second which means you at least do one swing of up and down in one second in Step 7. Otherwise, the results will not be accurate. Meanwhile, you can adjust the split time according to your own motion speed.

:::

- **Step 10.** Rebalance the dataset, Click **Dashboard** and drop down page to find **Perform train** / **test split**

Click Perform train / test split and choose Yes and confirm it

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition17.png" style={{width:800, height:'auto'}}/></div>

- **Step 11.** Create Impulse

Click **Create impulse** -> Add a processing block -> Choose **Spectral Analysis** -> Add a learning block -> Choose **Classification (Keras)** -> Save Impulse

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew1.png" style={{width:800, height:'auto'}}/></div>

- **Step 12.** Spectral features

Click and Set up

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew2.png" style={{width:800, height:'auto'}}/></div>

Click **Spectral features** -> Drop down page to click Save parameters -> Click **Generate features**

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew3.png" style={{width:800, height:'auto'}}/></div>

The output page should be like:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew4.png" style={{width:800, height:'auto'}}/></div>

- **Step 13.** Training your model

Click NN Classifier -> Click Start training -> Choose Unoptimized (float32)

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew5.png" style={{width:800, height:'auto'}}/></div>

- **Step 14.** Model testing

Click Model testing -> Click Classify all

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition23.png" style={{width:800, height:'auto'}}/></div>

:::note:

If your accuracy is low, you can check you dataset by increasing the training set and extending the sample time

:::

- **Step 15.** Build Arduino library

Click Deployment -> Click Arduino Library -> Click **Build** -> Download the .ZIP file

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew7.png" style={{width:400, height:'auto'}}/></div>

- **Step 16.** The name of .ZIP file is very important, it is set up as your name of the Edge Impulse project by default. Like here the project of the name is "XIAO-BLE-gestures_inferencing". Select the file as ""Add the ".ZIP file" to your Arduino libraries

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition35.png" style={{width:300, height:'auto'}}/></div>

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition36.png" style={{width:300, height:'auto'}}/></div>

- **Step 17.** Download the code [here](https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEI.ino). Change the name of your headfile as the name of your own and upload it.

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/Motion-Recognition33.png" style={{width:400, height:'auto'}}/></div>

- **Step 18.** Move or hold the Seeed Studio XIAO nRF52840 Sense and check the results:

Click the monitor on the top right corner of Arduino. 

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew15a.png" style={{width:800, height:'auto'}}/></div>

When you move the Seeed Studio XIAO nRF52840 Sense in the **left and right** direction:

The monitor will output something like:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew11a.png" style={{width:500, height:'auto'}}/></div>

And the output display is like:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew14a.png" style={{width:300, height:'auto'}}/></div>

When you move the Seeed Studio XIAO nRF52840 Sense in the **up and down** direction:

The monitor will output something like:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew9a.png" style={{width:500, height:'auto'}}/></div>

And the output display is like:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew12a.png" style={{width:300, height:'auto'}}/></div>

When you **hold** the Seeed Studio XIAO nRF52840 Sense in the idle state:

The monitor will output something like:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew10a.png" style={{width:500, height:'auto'}}/></div>

And the output display is like:

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/XIAOEInew13a.png" style={{width:300, height:'auto'}}/></div>

Congratulation! You acheve the end of the project. It is encouraged that you can try more directions and check which one will perform the best output.

## Resources

- [Seeed Studio XIAO nRF52840 Case File](https://files.seeedstudio.com/wiki/XIAO-BLE-Motion-Recognition/xiao-case-pink.stl)

## Tech support

Please submit any technical issues into our [forum](https://forum.seeedstudio.com/)

<p style={{textAlign:'center'}}><a href="https://www.seeedstudio.com/act-4.html?utm_source=wiki&utm_medium=wikibanner&utm_campaign=newproducts" target="_blank"><img src="https://files.seeedstudio.com/wiki/Wiki_Banner/new_product.jpg" /></a></p>
