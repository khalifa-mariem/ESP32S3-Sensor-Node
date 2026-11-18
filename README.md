# Autonomous Robot Telemetry & Health-Monitoring Node

A real-time safety and telemetry edge node built on the **ESP32-S3**. This system continuously monitors mobile robot health—acquiring 6-axis tilt/impact data and motor current consumption—applies digital low-pass filtering, and streams structured JSON telemetry over Wi-Fi/MQTT to a Python ground-station logger with real-time fault alerts.

---

## System Architecture

```text
[MPU6500 (6-Axis IMU)] ──(I2C)──┐
                                ├──► [ESP32-S3 MCU] ──(Wi-Fi / MQTT)──► [Python Ground Station]
[ACS712 (5A Current)]  ──(ADC)──┘    (C++ IIR Filter)                    (JSON Logger & Fault Alerts)
