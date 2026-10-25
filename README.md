# Distributed Dual-Pi Attire Compliance System

[![Hardware](https://img.shields.io/badge/Hardware-Raspberry%20Pi%205-C51A4A?style=for-the-badge&logo=raspberrypi&logoColor=white)](https://www.raspberrypi.com/)
[![Computer Vision](https://img.shields.io/badge/Computer%20Vision-YOLOv8%20%7C%20OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)](https://opencv.org/)
[![GUI](https://img.shields.io/badge/GUI-PyQt6-41CD52?style=for-the-badge&logo=qt&logoColor=white)](https://www.qt.io/)
[![Backend](https://img.shields.io/badge/Backend-Flask%20REST%20API-000000?style=for-the-badge&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![Academic Project](https://img.shields.io/badge/SIT-CEG1010%20Integrative%20Team%20Project-E31B23?style=for-the-badge)](https://www.singaporetech.edu.sg/)

An edge-AI physical access control station engineered to automate laboratory dress-code and personal protective equipment (PPE) compliance checks. The system classifies compliant attire (covered top, long pants, covered shoes) versus restricted garments (shorts, sleeveless shirts, open-toed footwear) using a dual-node distributed architecture across two Raspberry Pi 5 single-board computers, multi-threaded PyQt6 visual feedback, and zero-loss EXIF metadata audit logging over a localized network.

---

## 📌 Executive Summary

Manual attire enforcement at laboratory entry checkpoints is subjective, resource-intensive, and prone to inconsistent enforcement across staff shifts. Physical gatekeeping mechanisms such as turnstiles create high-traffic corridor bottlenecks and lack automated auditing capabilities.

The **Automated Computer Vision System** deploys a contactless, objective computer vision inspection station positioned directly at facility entrances:
* **Contactless Compliance Verification:** Analyzes approaching individuals within a calibrated Region of Interest (ROI) to verify safety compliance before displaying access status on the LCD.
* **Dual-Node Distributed Workload:** Separates real-time visual inferencing and UI rendering (Node 1: Edge Kiosk) from administrative database logging and OpenCV image cropping (Node 2: Server Hub) to eliminate CPU scheduling bottlenecks and thermal throttling.
* **Tamper-Resistant Audit Payloads:** Dynamically embeds bounding-box detection matrices as JSON strings inside EXIF metadata tags (`0x9286 UserComment`) of captured JPEG payloads, offloading image cropping to the backend server without edge latency penalties.

---
<p align="center">
  <img width="600" height="338" alt="hardware_demo" src="https://github.com/user-attachments/assets/9b55c926-a809-439d-bbf4-3f2253cbf235" alt="EdgeVision-AccessControl Live Detection Demo" width="680"/>
  <br>
  <em>Real-time YOLOv8 lab attire verification and Region of Interest (ROI) tracking running locally on Raspberry Pi 5.</em>
</p>

## 🏛️ System Architecture

```mermaid
graph TD
    subgraph NODE1 [NODE 1: EDGE KIOSK - RPi 5 + M12 Lens + 5in Touchscreen]
        direction TB
        camera[CSI Camera Module] -->|Zero-Copy DMA<br>Buffer Sharing| buffer[DMA Frame Buffer]
        buffer -->|pyqtSignal Dispatch| pyqt[PyQt6 Worker Thread QThread]
        pyqt -->|Real-Time Inference Loop| yolo[YOLOv8 Edge AI Engine]
        yolo --> detect{PPE Breach Detected?}
        detect -->|No - Valid Attire| access[Grant Door Access<br>Physical Relay / GPIO Action]
        detect -->|Yes - Infraction| exif[Inject JSON Bounding-Boxes<br>to EXIF Tag 0x9286 UserComment]
    end

    subgraph NETWORK [LOCAL LAN]
        exif -->|HTTP POST Request /upload<br>JPEG Payload with Injected EXIF| upload[Flask REST API Endpoint]
    end

    subgraph NODE2 [NODE 2: SERVER HUB - Secondary RPi 5 / Admin Host]
        direction TB
        upload --> extract[Extract EXIF Metadata<br>piexif Parser 0x9286]
        extract --> slice[OpenCV Matrix Slicing Engine]
        slice --> capture[Received_Captures/<br>Raw Full-Resolution JPEGs]
        slice --> crop[Cropped_People/<br>Isolated Infraction Crops]
    end

    style NODE1 fill:#161b22,stroke:#C51A4A,stroke-width:2px,color:#fff
    style NODE2 fill:#161b22,stroke:#41CD52,stroke-width:2px,color:#fff
    style NETWORK fill:#0d1117,stroke:#58a6ff,stroke-width:1px,stroke-dasharray: 5,color:#fff
    style detect fill:#b85c00,stroke:#ff9000,stroke-width:2px,color:#fff
