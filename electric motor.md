To adapt the previous project for controlling an electric motor instead of an electric bulb, you can follow a similar approach with modifications to accommodate the motor's specific requirements. Below is a detailed outline of the system, including hardware setup, programming, data collection, analysis, and predictive modeling.

Overview of the System

1. Microcontroller: To control the electric motor and collect data from sensors.


2. Sensors: To measure power consumption.


3. Database: To store collected data.


4. AI Models: For analyzing data and predicting lifespan and monthly electricity cost.


5. Laptop/Computer: For data processing, visualization, and model training.



Required Components

1. Microcontroller: Arduino, Raspberry Pi, or ESP8266/ESP32.


2. Power Consumption Sensor: ACS712 (for current) and a voltage sensor module.


3. Relay Module or Motor Driver: To control the electric motor (e.g., L298N motor driver for DC motors).


4. Electric Motor: DC motor suitable for your application.


5. Jumper Wires: For connections.


6. Breadboard: For prototyping.


7. Software Tools: Arduino IDE (for Arduino), Python (for data analysis).



1. Hardware Setup

1.1. Wiring Diagram

1. Connect the ACS712 current sensor:

VCC to Arduino 5V

GND to Arduino GND

OUT to an analog pin (e.g., A0)



2. Connect the voltage sensor:

Connect according to the sensor's instructions, usually to another analog pin (e.g., A1).



3. Connect the motor driver (e.g., L298N):

Connect the control pins to digital pins (e.g., D2 and D3) to control motor direction and speed.

Connect the motor terminals to the output pins of the motor driver.

Connect the motor driver VCC and GND to the power supply and Arduino.




1.2. Circuit Diagram

Here's a simple circuit diagram to visualize the connections:

+-----------+
         | Motor     |
         +-----------+
              |   |
              |   |
        +----------+
        |  Motor   |
        |  Driver  |
        +----------+
         |       |
         |       |
         |       |
         |    +--+
         |    |  |
         |    |AC|
         |    |  |
         |    +--+
         |       |
         |   ACS712
         |       |
         +-------+

2. Programming the Microcontroller

2.1. Arduino Code

Use the Arduino IDE to write a sketch that controls the motor and collects data from the sensors.

#include <SPI.h>
#include <Ethernet.h>

// Define pin numbers
const int motorPin1 = 2;  // Motor control pin 1
const int motorPin2 = 3;  // Motor control pin 2
const int currentPin = A0; // Current sensor pin
const int voltagePin = A1; // Voltage sensor pin

// Network setup (for Ethernet Shield)
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
EthernetServer server(80);

// Variables for power calculation
float current, voltage, power;

void setup() {
  Serial.begin(9600);
  pinMode(motorPin1, OUTPUT);
  pinMode(motorPin2, OUTPUT);
  
  // Start Ethernet
  Ethernet.begin(mac);
  server.begin();
}

void loop() {
  // Read current and voltage
  current = analogRead(currentPin) * (5.0 / 1023.0); // Convert ADC value to voltage
  voltage = analogRead(voltagePin) * (5.0 / 1023.0); // Same for voltage
  power = current * voltage; // Power in watts

  // Send data to the serial monitor
  Serial.print("Current: ");
  Serial.print(current);
  Serial.print(" A, Voltage: ");
  Serial.print(voltage);
  Serial.print(" V, Power: ");
  Serial.print(power);
  Serial.println(" W");

  // Control the motor
  digitalWrite(motorPin1, HIGH); // Rotate motor forward
  digitalWrite(motorPin2, LOW);
  delay(5000); // Run for 5 seconds
  digitalWrite(motorPin1, LOW); // Stop motor
  digitalWrite(motorPin2, LOW);
  delay(60000); // Wait for 1 minute before repeating

  // Send data to the laptop for processing (optional)
  // You can implement a method to send data over Ethernet or Serial.
}

3. Data Collection

You can choose to send data from the Arduino to a database or directly to a laptop over Serial or Ethernet. For simplicity, we will simulate saving data on the laptop side.

4. Database Setup

You can use SQLite for storing data.

4.1. Database Creation Script (Python)

Create a script that receives data from the microcontroller and stores it in an SQLite database.

import sqlite3

# Database setup
conn = sqlite3.connect('motor_data.db')
c = conn.cursor()
c.execute('''CREATE TABLE IF NOT EXISTS power_data (timestamp TEXT, current REAL, voltage REAL, power REAL)''')

# Function to insert data into the database
def insert_data(timestamp, current, voltage, power):
    c.execute("INSERT INTO power_data (timestamp, current, voltage, power) VALUES (?, ?, ?, ?)", (timestamp, current, voltage, power))
    conn.commit()

# Example of receiving data (replace this with your actual data reception logic)
while True:
    # Here you would read from the Serial or socket
    timestamp = "2024-10-09 12:00:00"  # Replace with actual timestamp
    current = 2.5  # Replace with actual data
    voltage = 230  # Replace with actual data
    power = current * voltage  # Calculate power
    insert_data(timestamp, current, voltage, power)

# Close the database connection when done
conn.close()

5. Data Analysis and Prediction

Use Python for data analysis and machine learning modeling.

5.1. Data Analysis Script

import pandas as pd
import sqlite3
import matplotlib.pyplot as plt
import seaborn as sns

# Load data from the database
conn = sqlite3.connect('motor_data.db')
data = pd.read_sql_query("SELECT * FROM power_data", conn)

# Convert timestamp to datetime
data['timestamp'] = pd.to_datetime(data['timestamp'])
data.set_index('timestamp', inplace=True)

# Visualize the power consumption
plt.figure(figsize=(10, 5))
sns.lineplot(data=data, x=data.index, y='power')
plt.title('Power Consumption Over Time')
plt.xlabel('Time')
plt.ylabel('Power (Watts)')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

# Close the database connection
conn.close()

5.2. Predictive Modeling

Use Scikit-Learn to create a machine learning model for predicting electricity costs based on power consumption.

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
import numpy as np

# Load data
conn = sqlite3.connect('motor_data.db')
data = pd.read_sql_query("SELECT * FROM power_data", conn)

# Calculate monthly cost (assume continuous operation)
data['cost'] = (data['power'] / 1000) * 24 * 30 * 0.1  # kWh cost

# Prepare data for modeling
X = data[['power']]
y = data['cost']

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Train the model
model = LinearRegression()
model.fit(X_train, y_train)

# Predictions
predictions = model.predict(X_test)

# Evaluate the model
mse = np.mean((y_test - predictions) ** 2)
print(f"Mean Squared Error: {mse:.2f}")

# Close the database connection
conn.close()

6. User Interface (Optional)

You could create a web interface using Flask to control the motor and display analytics, but this would require additional setup.

Final Steps

Testing: Test each part of the system incrementally (hardware, database, AI models).

Documentation: Document the wiring, code, and usage instructions.

Maintenance: Regularly monitor the system and update models as needed.


This setup provides a comprehensive way to control an electric motor, monitor power consumption, and analyze the data using a microcontroller and laptop. If you have any questions or need further details on specific parts, feel free to ask!

