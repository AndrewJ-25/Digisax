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
#include <map>
#include <initializer_list>

// =====================
// MATRIX PINS
// =====================

int Rows[] = {7, 15, 16, 17, 18, 8, 9};
int Columns[] = {10, 11, 12};

// =====================
// KEY NAMES
// =====================

const char* saxKey[7][3] =
{
    {"Octave1","L1","Ls1"},
    {"fork1","Bf1","Ls2"},
    {"Rs1","L2","Ls3"},
    {"Rs2","L3","Gsh1"},
    {"Rs3","R1","Csh1"},
    {"Dsh1","R2","LB1"},
    {"C1","R3","LBf1"}
};

// =====================
// BIT POSITIONS
// =====================

enum KeyBits
{
    OCTAVE1,
    L1,
    LS1,
    FORK1,
    BF1,
    LS2,
    RS1,
    L2,
    LS3,
    RS2,
    L3,
    GSH1,
    RS3,
    R1,
    CSH1,
    DSH1,
    R2,
    LB1,
    C1,
    R3,
    LBF1
};

// =====================
// KEY -> BIT LOOKUP
// =====================

std::map<String, uint8_t> keyToBit =
{
    {"Octave1", OCTAVE1},
    {"L1", L1},
    {"Ls1", LS1},
    {"fork1", FORK1},
    {"Bf1", BF1},
    {"Ls2", LS2},
    {"Rs1", RS1},
    {"L2", L2},
    {"Ls3", LS3},
    {"Rs2", RS2},
    {"L3", L3},
    {"Gsh1", GSH1},
    {"Rs3", RS3},
    {"R1", R1},
    {"Csh1", CSH1},
    {"Dsh1", DSH1},
    {"R2", R2},
    {"LB1", LB1},
    {"C1", C1},
    {"R3", R3},
    {"LBf1", LBF1}
};

// =====================
// FINGERING TABLE
// =====================

std::map<uint32_t, float> waves;

// =====================
// CREATE BITMASK
// =====================

uint32_t makeMask(std::initializer_list<uint8_t> keys)
{
    uint32_t mask = 0;

    for(auto key : keys)
    {
        mask |= (1UL << key);
    }

    return mask;
}

// =====================
// SETUP
// =====================

void setup()
{
    Serial.begin(115200);

    // Row pins output
    for(int i = 0; i < 7; i++)
    {
        pinMode(Rows[i], OUTPUT);
        digitalWrite(Rows[i], HIGH);
    }

    // Column pins input pullup
    for(int i = 0; i < 3; i++)
    {
        pinMode(Columns[i], INPUT_PULLUP);
    }

    // =====================
    // NOTE TABLE
    // =====================

    waves[makeMask({L1,L2,L3,R1,R2,R3,C1,LBF1})] = 138.59;
    waves[makeMask({L1,L2,L3,R1,R2,R3,C1,LB1})]  = 146.83;
    waves[makeMask({L1,L2,L3,R1,R2,R3,C1})]      = 155.56;
    waves[makeMask({L1,L2,L3,R1,R2,R3,C1,CSH1})] = 164.81;
    waves[makeMask({L1,L2,L3,R1,R2,R3})]         = 174.61;
    waves[makeMask({L1,L2,L3,R1,R2,R3,DSH1})]    = 185.00;
    waves[makeMask({L1,L2,L3,R1,R2})]            = 196.00;
    waves[makeMask({L1,L2,L3,R1})]               = 207.65;
    waves[makeMask({L1,L2,L3,R2})]               = 220.00;
    waves[makeMask({L1,L2,L3})]                  = 233.08;
    waves[makeMask({L1,L2,L3,GSH1})]             = 246.94;
    waves[makeMask({L1,L2})]                     = 261.63;
    waves[makeMask({L1,BF1})]                    = 277.18;
    waves[makeMask({L1})]                        = 293.66;
    waves[makeMask({L2})]                        = 311.13;
    waves[makeMask({})]                          = 329.63;

    Serial.println("Digital Sax Ready");
}

// =====================
// LOOP
// =====================

void loop()
{
    uint32_t currentMask = 0;

    // Scan all rows
    for(int row = 0; row < 7; row++)
    {
        // Disable all rows
        for(int r = 0; r < 7; r++)
        {
            digitalWrite(Rows[r], HIGH);
        }

        // Activate one row
        digitalWrite(Rows[row], LOW);

        delayMicroseconds(50);

        // Read columns
        for(int col = 0; col < 3; col++)
        {
            if(digitalRead(Columns[col]) == LOW)
            {
                const char* keyName = saxKey[row][col];

                currentMask |=
                    (1UL << keyToBit[String(keyName)]);

                Serial.print("Pressed: ");
                Serial.println(keyName);
            }
        }
    }

    // Lookup note
    auto note = waves.find(currentMask);

    if(note != waves.end())
    {
        Serial.print("Frequency: ");
        Serial.println(note->second);
    }
    else
    {
        Serial.println("Unknown fingering");
    }

    Serial.println("----------------");

    delay(50);
}
```

This code was written in C++ and I started by pulling all the column pins up by internal pull ups of the esp-32 and then i went with the getting all the row pins set to high and then started a loop where everytime i pull all the rows high and keep one row low and match that with the column if some column is low and has a bridge btw it and the row through a switch the code will print the row and the column on the serial monitor and also now it can detect multiple keys at once and map which note has been played and then according to that map which wave to be played i will refactor the waves and how play them when i build irl it's not possible now


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
