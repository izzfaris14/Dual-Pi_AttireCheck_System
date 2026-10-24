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

Manual dress-code and PPE inspections at laboratory entry points are labor-intensive, subjective, and prone to inconsistent enforcement. Physical turnstiles create high-traffic corridor bottlenecks and lack auditable compliance logs. 

**EdgeVision-AccessControl** deploys a contactless, objective computer vision inspection station positioned directly at facility entrances:
* **Real-Time Automated Inspection:** Identifies compliant attire (covered top, long pants, covered shoes) and restricted garments (shorts, sleeveless shirts, open-toed footwear).
* **Distributed Dual-Node Architecture:** Splits real-time visual inferencing (Node 1: Edge Kiosk) from logging, metadata extraction, and OpenCV matrix slicing (Node 2: Server Hub) to prevent thermal throttling and local memory exhaustion.
* **Fail-Safe Access Control:** Evaluates subject compliance within a defined Region of Interest (ROI) and maintains physical access locks by default unless safety requirements are met.
* **Tamper-Resistant Logging:** Dynamically embeds bounding-box detection matrices as JSON strings inside EXIF metadata tags (`0x9286 UserComment`) of captured JPEG payloads, offloading image cropping to the backend.

---

## 🏛️ System Architecture
## 🏛️ System Architecture

```mermaid
graph TD
    subgraph NODE1 [NODE 1: EDGE KIOSK]
        direction TB
        camera[CSI Camera Module] -->|Zero-Copy DMA| buffer[DMA Frame Buffer]
        buffer -->|Thread Signals| pyqt[PyQt6 Isolated QThread]
        pyqt -->|Inference Loop| yolo[YOLOv8 Edge AI Engine]
        yolo --> detect{Violation Detected?}
        detect -->|No| access[Grant Door Access<br>Relay / GPIO Action]
        detect -->|Yes| exif[Inject Bounding-Boxes to EXIF<br>Tag 0x9286]
    end

    subgraph NETWORK [LOCAL LAN]
        exif -->|HTTP POST Payload<br>JSON & JPEG| upload[Flask /upload API]
    end

    subgraph NODE2 [NODE 2: SERVER HUB]
        direction TB
        upload --> extract[Extract EXIF Metadata<br>Tag 0x9286]
        extract --> slice[OpenCV Matrix Slicing Engine]
        slice --> capture[Received_Captures<br>Raw Uncut JPEGs]
        slice --> crop[Cropped_Breaches<br>Isolated Violations]
    end

    style NODE1 fill:#1a1a24,stroke:#C51A4A,stroke-width:2px,color:#fff
    style NODE2 fill:#1a1a24,stroke:#41CD52,stroke-width:2px,color:#fff
    style NETWORK fill:#0d1117,stroke:#58a6ff,stroke-width:1px,stroke-dasharray: 5,color:#fff
    style detect fill:#b85c00,stroke:#ff9000,stroke-width:2px,color:#fff
---

## 📊 Technical Benchmarks & Model Evaluation

The system was evaluated by benchmarking two independently developed pipelines: a lightweight **YOLO PRO** model compiled into an Edge Impulse executable (`.eim`) and a production **YOLOv8** model running via ONNX runtime[cite: 49, 50, 64].

| Metric / Parameter | YOLO PRO (Edge Impulse `.eim`) | YOLOv8 Engine (Primary Deployment) | Engineering Resolution |
| :--- | :--- | :--- | :--- |
| **Inference Accuracy** | 67.0% mAP@50[cite: 11, 67] | **94.0% mAP (85%–95% range)**[cite: 39, 68] | Balanced class normalization across 1,458 images[cite: 16, 65, 68] |
| **Effective Range** | 3.44 meters[cite: 67, 69] | **5.65 meters**[cite: 68, 69] | 8mm M12 Portrait Lens calibrated at 40°–50° FOV[cite: 46, 47, 64] |
| **Small Object Recall** | 9.0% (Missed shoes/sandals)[cite: 12] | **73.0% – 90.0% Recall**[cite: 18, 68] | Pixel density focused at entrance threshold[cite: 37, 46] |
| **Execution Architecture** | Single-threaded sequential[cite: 72] | Multi-threaded `QThread`[cite: 20, 53] | UI rendering decoupled from inference pipeline[cite: 53] |
| **CPU Utilization** | Low single-core burst[cite: 20, 72] | 400% saturation (4 ARM cores)[cite: 20, 72] | Non-essential I/O & cropping offloaded to Node 2[cite: 40, 58, 71] |
| **Memory Footprint** | Stabilized at ~800 MB[cite: 20] | Pushed to ~950 MB[cite: 20] | Prevented OOM panics by eliminating container overhead[cite: 70, 72] |

---

## ⚙️ Key Hardware & Mechanical Innovations

* **CSI Zero-Copy Direct Memory Access (DMA):** Bypassed standard USB webcam latency by connecting the optical sensor directly over the Camera Serial Interface (CSI), enabling frames to pass into memory without CPU pipeline stalls[cite: 43, 46, 51].
* **Optical Calibration for Laboratory Hallways:** Benchmarked multiple lenses (wide-angle fisheye >80° vs telescopic <5°) and selected an **8mm 12MP Portrait Lens (40°–50° FOV)**[cite: 46, 47]. This established an optimal focal plane at 3.6m to capture full-body subjects without edge distortion[cite: 47, 64].
* **Custom 3D-Printed Articulating Chassis:** Designed in Autodesk Fusion and printed to solve display ribbon cable fragility[cite: 26, 48, 80]:
  * Solved flat ribbon cable bend limits by pivoting from a landscape clamshell to a vertical portrait form factor[cite: 26, 49, 81].
  * Engineered a dual-cylinder ratcheting hinge with a 12-step gear configuration, yielding **144 rigid viewing and camera angles** across a 270° sweep[cite: 27, 81].
  * Integrated an internal mounting frame accommodating the Raspberry Pi 5, Hailo-8L AI HAT+, and a 5-inch LAFVIN touchscreen display[cite: 48, 49, 61].

---

## 💻 Software Architecture & Network Pipeline

### 1. PyQt6 Multi-Threaded Decoupling (`Camera-detection.py`)
To prevent the high-load YOLOv8 CPU inference loop from locking the GUI, the capture and detection logic runs in an isolated `QThread`, emitting processed frames to the UI thread via `pyqtSignal`:
```python
class CameraThread(QThread):
    frame_ready = pyqtSignal(object)

    def run(self):
        while self.running:
            # AI Inference executes on an isolated background thread
            results = self.model(roi_frame, device='cpu')
            annotated_frame = results[0].plot()
            # Safely transfer frame to the primary UI event loop
            self.frame_ready.emit(annotated_frame)
