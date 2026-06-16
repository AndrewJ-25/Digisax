# Digisax WORK IN PROGRESS!

Will be completed before the end of June

ESP32 controlled wind synth

# Schematic

![Alt text](images/digisax_schematic.jpg)
See images/digisax_schematic.pdf for better resolution image

# Description
- ESP32-S3-DevKitC-1 microcontroller
- keys arranged in a matrix of 3 Columns * 7 Rows
- breath sensor consisting of MPXV7007 differential pressure sensor + mcp3221 ADC to communicate over I2C
- rotary encoder to adjust volume
- headphone output via adafruit tlv320dac3100 DAC, controlled with I2C and I2S https://learn.adafruit.com/adafruit-tlv320dac3100-i2s-dac

# Code 

```cpp
#include <Arduino.h>

 int Rows[] = {7,15,16,17,18,8,9}; // defined a row array with all the row pins
 int Columns[] = {10,11,12}; // defined a column array with all the column pins 

 void setup()
 {
    // keep in mind that the columns needs to be high until there is a bridge between them and the row which is made by the switch if they are not pulled up    externally i will do that now
   
  // setting all the row pins high

   for(int i=0; i<7 ; i++){  // this loop run for all the row pins 
    int Rowpin = Rows[i];

    pinMode(Rowpin , OUTPUT);
    digitalWrite(Rowpin , HIGH); 
   }

   // giving all the columns an internal pull up resistor

   for (int j = 0; j<3; j++) { // this loop runs for all column pin 
        int Colpin = Columns[j];

        pinMode(Colpin , INPUT_PULLUP);
   }

   // starting the serial montior 
   Serial.begin(115200);
}

void loop()
{

     // setting all the row pins high
   for(int m = 0; m<7 ; m++){ // this is the main loop
      for(int i=0; i<7 ; i++){  // this loop run for all the row pins and sets them high
          int Rowpin = Rows[i];

          pinMode(Rowpin , OUTPUT);
          digitalWrite(Rowpin , HIGH); 
      }

      // setting a pin low so that we can scan the matrix
      int LowRowPin = Rows[m];
      pinMode(LowRowPin , OUTPUT);
      digitalWrite(LowRowPin , LOW);
      delayMicroseconds(50);

      for(int i=0; i<3 ; i++){  // this loop run for all the row pins 
          int Columnpin = Columns[i];
          if (digitalRead(Columnpin) == LOW){
            Serial.println("You Pressed");
            Serial.println("Row");
            Serial.println(m);
            Serial.println("Column");
            Serial.println(i);
          }
      }
   }

   

      delay(10);
   // had to set all the rows high again cause when we will scan all the rows if we do row1 low and scan and dont get a key and move onto row2 and set it low row 1 also remains low because the setup() code works once
  
}
```

This code was written in C++ and I started by pulling all the column pins up by internal pull ups of the esp-32 and then i went with the getting all the row pins set to high and then started a loop where everytime i pull all the rows high and keep one row low and match that with the column if some column is low and has a bridge btw it and the row through a switch the code will print the row and the column on the serial monitor 


## BOM
|Quantity|Description|Identifier|Source|
|-|-|-|-|
|1|DAC amplifier|adafruit tlv320dac3100 breakout board|-|
|1|MCU dev-board|ESP32-S3-DevKitC-1|-|
|1|Differential pressure sensor|MPXV7007DP|-|
|1|I2C ADC|mcp3221|
|-|Keyboard switches|Cherry MX Black|-|
|1|0.91" OLED display|-|-|
|1|rotary encoder|EC11|-|
