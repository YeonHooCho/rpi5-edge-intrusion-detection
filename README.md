# Containerized Edge Intrusion Detection & Real-Time Telemetry Pipeline (Raspberry Pi 5)

> **Raspberry Pi 5 (ARM64) · ADXL345 (I2C/SPI) · Docker & Nginx · Python Flask · SQLite3**  
> An edge-computing physical intrusion monitoring system engineered on Raspberry Pi 5, combining hardware-level signal filtering, isolated Docker container infrastructure, and asynchronous mobile web polling to achieve **sub-0.5s end-to-end alert latency**.

---

## Overview & Engineering Problem Solving
- **Hardware Fault Diagnosis & Pivot:** During initial prototyping with an analog sensor and ADC (`MCP3002`), bench oscilloscope and multimeter diagnostics revealed severe signal noise and sample drops. Re-architected the hardware interface to a digital 3-axis accelerometer (**ADXL345**) over the **I2C** bus for deterministic digital telemetry.
- **Gravity-Offset Filtering Algorithm:** Eliminated continuous false positives caused by static Earth gravity ($9.8\text{ m/s}^2$) along the Z-axis by applying real-time baseline compensation ($Z_{\text{filtered}} = Z_{\text{raw}} - 9.8$) with threshold event detection ($\vert{}Z_{\text{filtered}}\vert{} > 3.0$).
- **ARM64 Infrastructure Isolation:** Resolved ARM64 container compatibility bottlenecks by decoupling lightweight local time-series persistence (**SQLite3**) from an isolated **Docker + Nginx** web gateway, ensuring zero host downtime even under peripheral I/O faults.

## System Architecture

```text
[ADXL345 3-Axis Accelerometer] ──(I2C Bus)──▶ [Raspberry Pi 5 Edge Collector (insert_gyro.py)]
                                                       │
                                           (Gravity Compensation & Filter)
                                                       ▼
                                           [SQLite3 Embedded DB (smartroom.db)]
                                                       │
                                              (Flask REST API: /api/data)
                                                       ▼
                                    [Docker Container: Nginx Reverse Proxy]
                                                       │
                                          (Wi-Fi / 500ms Async Fetch Loop)
                                                       ▼
                                    [Mobile Web Dashboard & Red-Alert UI]
