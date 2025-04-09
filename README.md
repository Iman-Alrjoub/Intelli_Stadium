# Smart Sustainable Stadium – Arduino Prototype

## 📌 Idea Overview
This project demonstrates a prototype for an energy-generating smart stadium using Arduino technology. It focuses on converting kinetic and sound energy into usable electrical energy inside sports facilities.

## ⚙️ What This Code Does
The Arduino sketch reads input from:
- Pressure sensors on smart floor tiles (representing audience movement)
- Sound sensors capturing crowd noise
- Optional: Vibration sensors simulating ball movement

It processes the signals and simulates energy generation based on input intensity.

## 🔧 Tools and Components
- Arduino Uno
- Pressure sensors
- Sound sensor module
- Breadboard and jumper wires
- LED or display to indicate generated energy (simulated)

## 🧪 How to Run
1. Connect components as shown in the circuit diagram (to be added).
2. Upload `.ino` file to Arduino board using Arduino IDE.
3. Move or clap near the sensors to simulate input and observe LED or serial output.

## 🧠 Future Enhancements
- Integration with real-time dashboards
- Energy storage simulation
- Integration with AI for optimization

# Intelli_Stadium
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_NeoPixel.h>

#define SCREEN_WIDTH 128  // OLED display width, in pixels
#define SCREEN_HEIGHT 64  // OLED display height, in pixels
#define OLED_RESET    -1  // Reset pin for OLED
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define NEOPIXEL_PIN   6       // Pin to which the NeoPixel (WS2812B) data line is connected
#define NUM_PIXELS     37      // Number of LEDs in the strip
Adafruit_NeoPixel strip(NUM_PIXELS, NEOPIXEL_PIN, NEO_GRB + NEO_KHZ800);  // NeoPixel object

int piezoPin = A0;          // Pin connected to the first piezoelectric sensor
int piezoPin2 = A1;         // Pin connected to the second piezoelectric sensor (for counting kicks)
int ledPin = LED_BUILTIN;   // Built-in LED pin (or use any other pin for an external LED)

float thresholdVoltage = 1;  // Voltage threshold to trigger blinking (adjustable)
float thresholdVoltage2 = 0.37;  // Voltage threshold to trigger blinking sensor 2 (adjustable)
int sensorValue = 0;           // Variable to store sensor value for the first piezoelectric sensor
int sensorValue2 = 0;          // Variable to store sensor value for the second piezoelectric sensor

int kickCount = 0;            // Count the number of kicks

// Set the color to white (255, 255, 255)
uint32_t whiteColor = strip.Color(255, 255, 255);

void setup() {
  Serial.begin(9600);

  // Initialize OLED display
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {  // 0x3C is the default I2C address
    Serial.println(F("SSD1306 allocation failed"));
    for (;;);  // Don't proceed if the OLED display doesn't initialize
  }
  display.display();
  delay(2000);

  // Initialize NeoPixel strip
  strip.begin();
  strip.show();  // Initialize all pixels to 'off'

  // Initialize piezo pins as input
  pinMode(piezoPin, INPUT);
  pinMode(piezoPin2, INPUT);
  pinMode(ledPin, OUTPUT);  // Initialize the simple LED pin as an output
}

void loop() {
  // Read the analog value from the first piezoelectric sensor
  sensorValue = analogRead(piezoPin);
  // Map the sensor value to a voltage (0-1023 mapped to 0-5V)
  float voltage = 10 * (sensorValue * 5.0) / 1023.0;

  // Display the voltage and kick count on the OLED screen
  display.clearDisplay();
  
  // Set text size for voltage display to a smaller size
  display.setTextSize(1); // Smaller text size for voltage
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.print("Voltage: ");
  display.setTextSize(2);  // Larger text for the voltage value
  display.setCursor(0, 10); // Adjusted position for the voltage value
  display.print(voltage, 2);
  display.print(" V");

  // Display the kick count
  display.setTextSize(2);  // Larger text for the kick count
  display.setCursor(0, 40);  // Move down to display the kick count
  display.print("Kicks: ");
  display.print(kickCount);  // Show the kick count

  // Show the updated display
  display.display();

  // If the voltage from the first sensor exceeds the threshold, trigger blinking with white color
  if (voltage > thresholdVoltage) {
    // Blink LEDs in white color
    for (int i = 0; i < NUM_PIXELS; i++) {
      strip.setPixelColor(i, whiteColor);  // Set LEDs to white color
    }
    strip.show();  // Update the strip to turn on the LEDs
    delay(500);  // Keep LEDs on for 500ms (half a second)

    // Turn off the LEDs
    for (int i = 0; i < NUM_PIXELS; i++) {
      strip.setPixelColor(i, strip.Color(0, 0, 0));  // Turn off LEDs
    }
    strip.show();  // Update the strip to turn off the LEDs
    delay(500);  // Keep LEDs off for 500ms (half a second)
  } else {
    // If the voltage from the first sensor is below the threshold, keep the LEDs off
    for (int i = 0; i < NUM_PIXELS; i++) {
      strip.setPixelColor(i, strip.Color(0, 0, 0));  // Turn off LEDs
    }
    strip.show();  // Update the strip
  }

  // Read the analog value from the second piezoelectric sensor (for counting kicks)
  sensorValue2 = analogRead(piezoPin2);
  // Map the sensor value to a voltage (0-1023 mapped to 0-5V)
  float voltage2 = 10 * (sensorValue2 * 5.0) / 1023.0;

  // If the voltage from the second sensor exceeds the threshold, count a kick
  if (voltage2 > thresholdVoltage2) {
    digitalWrite(ledPin, HIGH);  // Turn on the simple LED when the second sensor is triggered
    kickCount++;  // Increment the kick count
    Serial.print("Kick count: ");
    Serial.println(kickCount);  // Print the kick count to the serial monitor
    delay(500);  // Delay to debounce the sensor (avoid multiple counts for a single touch)
    digitalWrite(ledPin, LOW);  // Turn off the simple LED after the kick
  }

  delay(100);  // Small delay for stability
}
