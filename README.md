# Water_Flow_Monitoring_System

# **Water Flow Monitoring System with Arduino and OLED Display**

This project involves setting up a water flow monitoring system using an Arduino Uno, a water flow rate sensor, and an OLED display. The system will display the real-time water flow rate on the OLED display.


## **Introduction**
This project demonstrates how to monitor water flow using an Arduino Uno and display the flow rate on an OLED screen. The system measures the water flow rate using a flow sensor and calculates the flow rate, displaying the result in real-time.

## **Components Used**
- **Arduino Uno**: The microcontroller used to read the sensor data and control the OLED display.
- **Water Flow Rate Sensor**: Measures the rate of water flow through a pipe.
- **OLED Display (I2C)**: Displays the water flow rate.
- **Jumper Wires**: making connections.

## **Hardware Setup**
### **Connections**
1. **Water Flow Rate Sensor**:
   - **VCC**: Connect to the 5V pin on the Arduino.
   - **GND**: Connect to the GND pin on the Arduino.
   - **Signal**: Connect to digital pin 2 on the Arduino.

2. **OLED Display**:
   - **VCC**: Connect to the 3.3V pin on the Arduino.
   - **GND**: Connect to the GND pin on the Arduino.
   - **SCL**: Connect to the A5 pin on the Arduino.
   - **SDA**: Connect to the A4 pin on the Arduino.



## **Software Setup**
### **Arduino Libraries**
1. Install the necessary libraries in the Arduino IDE:
   - `Adafruit_GFX`
   - `Adafruit_SSD1306`
   - `Wire`

### **Arduino Code**
Upload the following code to your Arduino Uno:

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 32 // OLED display height, in pixels

#define OLED_RESET     -1 // Reset pin # (or -1 if sharing Arduino reset pin)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

const int flowSensorPin = 2; // Pin where the flow sensor is connected
volatile int flowPulseCount = 0;
unsigned long oldTime = 0;

float calibrationFactor = 4.5; // Calibration factor for the specific flow sensor
float flowRate = 0;
float flowMilliLitres = 0;
float totalMilliLitres = 0;

void setup() {
  // Initialize serial communication
  Serial.begin(9600);

  // Initialize the OLED display
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // Address 0x3C for 128x32
    Serial.println(F("SSD1306 allocation failed"));
    for(;;);
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  display.println("Water Flow Rate:");
  display.display();

  // Initialize the flow sensor
  pinMode(flowSensorPin, INPUT);
  digitalWrite(flowSensorPin, HIGH); // Optional internal pull-up
  attachInterrupt(digitalPinToInterrupt(flowSensorPin), pulseCounter, FALLING);
  
  // Set the initial time
  oldTime = millis();
}

void loop() {
  if ((millis() - oldTime) > 1000) { // Only process counters once per second
    detachInterrupt(digitalPinToInterrupt(flowSensorPin));

    // Calculate the flow rate in litres/min
    flowRate = ((1000.0 / (millis() - oldTime)) * flowPulseCount) / calibrationFactor;

    // Reset the pulse counter
    flowPulseCount = 0;

    // Store the old time
    oldTime = millis();

    // Calculate the flow in millilitres
    flowMilliLitres = (flowRate / 60) * 1000;

    // Calculate the total flow in millilitres
    totalMilliLitres += flowMilliLitres;

    // Print the flow rate to the serial monitor
    Serial.print("Flow rate: ");
    Serial.print(flowRate);
    Serial.println(" L/min");

    // Print the total accumulated flow to the serial monitor
    Serial.print("Total: ");
    Serial.print(totalMilliLitres);
    Serial.println(" mL");

    // Display the flow rate on the OLED
    display.clearDisplay();
    display.setCursor(0,0);
    display.println("Water Flow Rate:");
    display.print(flowRate);
    display.println(" L/min");
    display.display();

    // Re-enable the interrupt
    attachInterrupt(digitalPinToInterrupt(flowSensorPin), pulseCounter, FALLING);
  }
}

// Interrupt Service Routine
void pulseCounter() {
  flowPulseCount++;
}



