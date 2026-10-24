# EdgeVision-AccessControl: Automated Dual-Pi Attire Check System

[![Platform](https://img.shields.io/badge/Platform-Raspberry%20Pi%205-C51A4A?style=flat&logo=raspberrypi&logoColor=white)](https://www.raspberrypi.com/)
[![Framework](https://img.shields.io/badge/Deep%20Learning-YOLOv8%20%7C%20Edge%20Impulse-blue?style=flat)](https://ultralytics.com/)
[![GUI](https://img.shields.io/badge/GUI-PyQt6%20%7C%20OpenCV-green?style=flat&logo=qt)](https://www.qt.io/)
[![Backend](https://img.shields.io/badge/Backend-Python%20Flask-black?style=flat&logo=flask)](https://flask.palletsprojects.com/)
[![License](https://img.shields.io/badge/Academic%20Project-SIT%20CEG1010-orange?style=flat)](https://www.singaporetech.edu.sg/)

An industrial-grade, distributed edge-AI physical access control station built on Raspberry Pi 5 single-board computers. The system automates laboratory dress code enforcement (detecting lab-compliant footwear, pants, and shirts versus prohibited shorts, sleeveless tops, and open-toed shoes) using an optimized YOLOv8 computer vision pipeline, real-time PyQt6 feedback, and automated audit logging over a localized client-server network.

Developed as the **CEG1010 Integrative Team Project (ITP)** by Group ID 8 at the **Singapore Institute of Technology (SIT)**.

---

## 📌 Executive Summary

Manual dress-code and PPE inspections at laboratory entry points are labor-intensive, subjective, and prone to inconsistent enforcement[cite: 37, 38]. Physical turnstiles create high-traffic corridor bottlenecks and lack auditable compliance logs. 

**EdgeVision-AccessControl** deploys a contactless, objective computer vision inspection station positioned directly at facility entrances:
* **Real-Time Automated Inspection:** Identifies compliant attire (covered top, long pants, covered shoes) and restricted garments (shorts, sleeveless shirts, open-toed footwear).
* **Distributed Dual-Node Architecture:** Splits real-time visual inferencing (Node 1: Edge Kiosk) from logging, metadata extraction, and OpenCV matrix slicing (Node 2: Server Hub) to prevent thermal throttling and local memory exhaustion.
* **Fail-Safe Access Control:** Evaluates subject compliance within a defined Region of Interest (ROI) and maintains physical access locks by default unless safety requirements are met.
* **Tamper-Resistant Logging:** Dynamically embeds bounding-box detection matrices as JSON strings inside EXIF metadata tags (`0x9286 UserComment`) of captured JPEG payloads, offloading image cropping to the backend.

---

## 🏛️ System Architecture
