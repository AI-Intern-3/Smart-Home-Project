Designing an electric system to measure power consumption using voltage and current sensors involves several components and steps. The primary goal is to continuously monitor voltage and current to calculate power consumption in real-time. Here’s a detailed guide on how to build such a system.

1. System Overview

The system will:

Measure Voltage and Current: Use sensors to measure voltage and current in an electrical circuit.

Calculate Power Consumption: Use the measured voltage and current to calculate power (in watts).

Log Data: Store measured data for analysis.

Display Data: Optionally, display real-time measurements on a screen.


2. Components Required

Hardware

Microcontroller: (e.g., Arduino or Raspberry Pi) for processing sensor data.

Voltage Sensor: (e.g., ZMPT101B) for measuring AC voltage.

Current Sensor: (e.g., ACS712) for measuring AC or DC current.

LCD Display (Optional): To display real-time voltage, current, and power readings.

Power Supply: For the microcontroller and sensors.

Resistors: As needed for sensor connections.


Software

Programming Language: Arduino IDE (for Arduino) or Python (for Raspberry Pi).

Database: SQLite or MySQL for logging data.


3. System Architecture

The architecture includes:

1. Sensor Layer: Voltage and current sensors connected to the microcontroller.


2. Processing Layer: The microcontroller processes the sensor data to calculate power consumption.


3. Data Logging Layer: Stores sensor readings in a database.


4. Display Layer (Optional): Displays real-time measurements on an LCD.



4. Implementation Steps

Step 1: Set Up Sensors

Wiring the Voltage Sensor:

Connect the voltage sensor (ZMPT101B) to the AC line you want to measure.

Follow the sensor's datasheet for proper connections.


Wiring the Current Sensor:

Connect the current sensor (ACS712) in series with the load.

Ensure correct polarity to avoid damage.



Step 2: Write the Monitoring Code

For Arduino: Here's an example Arduino code to measure voltage and current:


#include <LiquidCrystal.h>

// Constants
const int voltagePin = A0;   // Voltage sensor connected to analog pin A0
const int currentPin = A1;   // Current sensor connected to analog pin A1

// Create LCD object
LiquidCrystal lcd(12, 11, 5, 4, 3, 2); // Change pins as per your connection

// Calibration constants
const float voltageCalibration = 5.0 / 1023.0; // Voltage calibration for Arduino
const float currentCalibration = 5.0 / 1023.0; // Current calibration for Arduino

void setup() {
  lcd.begin(16, 2); // Initialize the LCD
  Serial.begin(9600); // Start serial communication
}

void loop() {
  // Read voltage
  int voltageReading = analogRead(voltagePin);
  float voltage = voltageReading * voltageCalibration * (220.0 / 5.0); // Adjust for your voltage (220V in this case)

  // Read current
  int currentReading = analogRead(currentPin);
  float current = currentReading * currentCalibration; // Assuming a current sensor range of 0-5V

  // Calculate power (P = V * I)
  float power = voltage * current; // Power in Watts

  // Print readings to Serial Monitor
  Serial.print("Voltage: ");
  Serial.print(voltage);
  Serial.print(" V, Current: ");
  Serial.print(current);
  Serial.print(" A, Power: ");
  Serial.print(power);
  Serial.println(" W");

  // Display readings on LCD
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("V: " + String(voltage) + "V");
  lcd.setCursor(0, 1);
  lcd.print("I: " + String(current) + "A");

  delay(1000); // Update every second
}

Step 3: Data Logging

To log the data in a database, you can modify the code to include a database connection (for Raspberry Pi) or write data to an SD card (for Arduino).

For Raspberry Pi (Python):

You can use the following Python script to read from the sensors and log data:

import RPi.GPIO as GPIO
import time
import sqlite3

# Setup GPIO
GPIO.setmode(GPIO.BCM)

# Sensor pins
voltage_pin = 17  # Change as needed
current_pin = 18  # Change as needed

# Connect to SQLite Database
conn = sqlite3.connect('power_consumption.db')
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS power_data
             (id INTEGER PRIMARY KEY, timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
              voltage REAL, current REAL, power REAL)''')
conn.commit()

def read_voltage():
    # Add your voltage reading logic here
    return voltage  # Replace with actual reading

def read_current():
    # Add your current reading logic here
    return current  # Replace with actual reading

try:
    print("System is ready.")
    while True:
        voltage = read_voltage()
        current = read_current()

        # Calculate power consumption
        power = voltage * current  # Power in Watts

        # Log data to database
        c.execute("INSERT INTO power_data (voltage, current, power) VALUES (?, ?, ?)", 
                  (voltage, current, power))
        conn.commit()

        print(f"Voltage: {voltage} V, Current: {current} A, Power: {power} W")
        
        time.sleep(1)  # Read every second

except KeyboardInterrupt:
    print("System shutting down.")
finally:
    conn.close()
    GPIO.cleanup()

5. Testing the System

Test Each Sensor: Measure known loads (like a light bulb) to verify the accuracy of the voltage and current readings.

Check Data Logging: Ensure that the data is being logged correctly to the database.


6. Deployment Considerations

Safety: Ensure proper safety measures when working with AC mains voltage.

Calibration: Calibrate sensors for accuracy.

Internet Connectivity: If using a Raspberry Pi, you can extend the project to send data to a cloud service for remote monitoring.


7. Enhancements

Data Analysis: Use the logged data to perform analysis on power consumption trends over time.

Web Interface: Create a web dashboard to visualize the data in real-time.

Predictive Modeling: Implement AI/ML models to predict future power consumption based on historical data.


Conclusion

This power consumption measurement system effectively utilizes voltage and current sensors to monitor and log power consumption. The data can be used for analysis and predictive modeling, helping to optimize energy usage. If you have any further questions or need additional assistance, feel free to ask!

