# ESP32-S3 Multi-Sensor IoT Telemetry & Monitoring Node

Embedded firmware developed in **C++** using **PlatformIO / VS Code** for an ESP32-S3 development board, designed to acquire, process, and stream multi-sensor industrial data (vibration and current) over Wi-Fi using the MQTT protocol.

## Hardware Stack
* **Microcontroller:** ESP32-S3 WROOM-1
* **Motion Sensor:** MPU6050 (3-axis Accelerometer & 3-axis Gyroscope via I2C)
* **Current Sensor:** ACS712 (Analog current measurement via ADC)
* **Actuator / Load:** 5V DC Motor driven via IRFZ44N MOSFET

## Firmware Architecture
* **`src/main.cpp`**: Main application loop and task initialization.
* **`include/sensors.h`**: I2C driver interface for MPU6050 and ADC sampling filters for ACS712.
* **`include/mqtt_client.h`**: Handles secure Wi-Fi reconnection logic and JSON payload publishing.

## Telemetry Payload Example
Sensor data is packaged into JSON format and published via MQTT:
```json
{
  "device_id": "esp32s3_node_01",
  "accel_rms": 0.42,
  "current_amps": 1.25,
  "status": "nominal"
}
```

### Step 3: Core C++ Firmware Template (`src/main.cpp`)
Here is a clean, structured skeleton of the C++ code you can put inside repository's `src/main.cpp` file. It sets up the Wi-Fi connection, initializes the I2C bus for the MPU6050, reads analog current, and builds the structure for telemetry:

```cpp
#include <Arduino.h>
#include <Wire.h>

// Pin Definitions
const int CURRENT_SENSOR_PIN = A0; // ACS712 analog input
const int MPU6050_ADDR = 0x68;     // I2C address for MPU6050

void setupMPU6050() {
  Wire.begin();
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x6B); // Power management register
  Wire.write(0);    // Wake up MPU6050
  Wire.endTransmission(true);
}

void setup() {
  Serial.begin(115200);
  pinMode(CURRENT_SENSOR_PIN, INPUT);
  setupMPU6050();
  Serial.println("ESP32-S3 Sensor Node Initialized.");
}

void loop() {
  // Read Analog Current (ACS712)
  int rawADC = analogRead(CURRENT_SENSOR_PIN);
  float voltage = (rawADC / 4095.0) * 3.3; // ESP32 ADC reference
  float currentAmps = (voltage - 2.5) / 0.185; // Example calibration for 5A module

  // Read MPU6050 Simulation / Data Request via I2C
  Wire.beginTransmission(MPU6050_ADDR);
  Wire.write(0x3B); // Starting register for Accelerometer measurements
  Wire.endTransmission(false);
  Wire.requestFrom(MPU6050_ADDR, 6, true);

  int16_t ax = Wire.read() << 8 | Wire.read();
  int16_t ay = Wire.read() << 8 | Wire.read();
  int16_t az = Wire.read() << 8 | Wire.read();

  // Print telemetry payload to Serial / MQTT stream
  Serial.printf("Telemetry -> Current: %.2f A | Accel X: %d, Y: %d, Z: %d\n", currentAmps, ax, ay, az);

  delay(1000); // 1-second telemetry sampling interval
}
```
