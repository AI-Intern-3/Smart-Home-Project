To implement a project for controlling an electric bulb, collecting power consumption data using sensors, storing the data in a database, and analyzing it with AI models using a microcontroller (like Arduino or Raspberry Pi), follow these steps:

Overview of the System

1. Microcontroller: To control the electric bulb and collect data from sensors.


2. Sensors: To measure power consumption.


3. Database: To store collected data.


4. AI Models: For analyzing data and predicting lifespan and monthly light bills.


5. Laptop/Computer: For data processing, visualization, and model training.



Required Components

1. Microcontroller: Arduino, Raspberry Pi, or ESP8266/ESP32.


2. Power Consumption Sensor: ACS712 (for current) and voltage sensor module.


3. Relay Module: To control the electric bulb.


4. Jumper Wires: For connections.


5. Breadboard: For prototyping.


6. Laptop/Computer: For data processing.


7. Software Tools: Arduino IDE (for Arduino), Python (for data analysis).



1. Hardware Setup

1.1. Wiring Diagram

1. Connect the ACS712 sensor:

VCC to Arduino 5V

GND to Arduino GND

OUT to an analog pin (e.g., A0)



2. Connect the voltage sensor:

Connect according to the sensor's instructions, usually to another analog pin (e.g., A1).



3. Connect the relay module:

VCC to Arduino 5V

GND to Arduino GND

Connect the relay input to a digital pin (e.g., D2) to control the bulb.




1.2. Circuit Diagram

Here’s a simple circuit diagram to visualize the connections:

+---------+
         |  Relay  |
         +---------+
            |   |
            |   |
        Bulb --  AC
            |   |
         +--+   +--+
         |           |
        (Power)  (Sensor)
         |           |
         |  ACS712   |
         |           |
         +-----------+

2. Programming the Microcontroller

2.1. Arduino Code

Use the Arduino IDE to write a sketch that controls the relay and collects data from the sensors.

#include <SPI.h>
#include <Ethernet.h>

// Define pin numbers
const int relayPin = 2;   // Relay control pin
const int currentPin = A0; // Current sensor pin
const int voltagePin = A1; // Voltage sensor pin

// Network setup (for Ethernet Shield)
byte mac[] = { 0xDE, 0xAD, 0xBE, 0xEF, 0xFE, 0xED };
EthernetServer server(80);

// Variables for power calculation
float current, voltage, power;

void setup() {
  Serial.begin(9600);
  pinMode(relayPin, OUTPUT);
  digitalWrite(relayPin, LOW); // Start with the relay off

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

  // Control the relay based on some condition (example: switch on for 5 seconds)
  digitalWrite(relayPin, HIGH); // Turn on the bulb
  delay(5000);
  digitalWrite(relayPin, LOW); // Turn off the bulb
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
conn = sqlite3.connect('bulb_data.db')
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
conn = sqlite3.connect('bulb_data.db')
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

Use Scikit-Learn to create a machine learning model for predicting light bills.

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
import numpy as np

# Load data
conn = sqlite3.connect('bulb_data.db')
data = pd.read_sql_query("SELECT * FROM power_data", conn)

# Calculate monthly bill (assume continuous operation)
data['bill'] = (data['power'] / 1000) * 24 * 30 * 0.1  # kWh cost

# Prepare data for modeling
X = data[['power']]
y = data['bill']

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

You could create a web interface using Flask to control the bulb and display analytics, but this would require additional setup.

Final Steps

Testing: Test each part of the system incrementally (hardware, database, AI models).

Documentation: Document the wiring, code, and usage instructions.

Maintenance: Regularly monitor the system and update models as needed.


This setup provides a comprehensive way to control an electric bulb, monitor power consumption, and analyze the data using a microcontroller and laptop. If you have any questions or need further details on specific parts, feel free to ask!

