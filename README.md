# QR-AR: Augmented Reality with QR Codes

A computer vision and augmented reality project that uses **QR code detection, camera pose estimation, and 3D projection** to create real-time augmented reality experiences.

The system detects a QR code through a webcam and uses its position and orientation to either:

* Project a **3D cube** onto the QR code
* Project a **3D pyramid** onto the QR code
* Replace the QR code region with **real-time video content**

The project demonstrates how computer vision can bridge the gap between a physical marker and digital augmented content.

---

## Features

### 1. 3D Cube Projection

Detects a QR code using the webcam and estimates its 3D orientation using OpenCV's `solvePnP()` algorithm.

A virtual cube is then projected onto the QR code using `cv2.projectPoints()`.

**Pipeline:**

`Webcam → QR Detection → Corner Extraction → Pose Estimation → 3D Projection → Augmented Frame`

---

### 2. 3D Pyramid Projection

The second module extends the same augmented-reality pipeline to project a **3D pyramid** onto the detected QR code.

The QR code acts as a planar reference surface, while the pyramid is constructed using 3D coordinates and projected into the camera's perspective.

**Pipeline:**

`Webcam → QR Detection → Pose Estimation → Pyramid Projection → Augmented Frame`

---

### 3. Video Overlay

The third module turns the QR code into a **video trigger**.

When a QR code is detected, a video is played inside the QR code's bounding region, creating the effect of digital content appearing directly on the physical marker.

**Pipeline:**

`Webcam → QR Detection → Bounding Box → Video Frame → Resize → Overlay`

---

## Technologies Used

* **Python**
* **OpenCV** — computer vision, camera processing and 3D projection
* **NumPy** — numerical operations and 3D coordinate manipulation
* **PyZBar** — QR code detection and decoding
* **PyQt5** — real-time graphical interface
* **Webcam** — live image acquisition

---

## Project Structure

```text
QR-AR/
│
├── cube.py
├── pyramid.py
├── video_overlay.py
├── video/
│   └── video.mp4
│
├── README.md
└── requirements.txt
```

The filenames can be changed depending on how the project is organized.

---

## How It Works

### Step 1 — Capture Video

OpenCV accesses the computer's webcam using:

```python
cv2.VideoCapture(0)
```

Frames are continuously captured from the camera.

---

### Step 2 — Detect the QR Code

PyZBar scans each camera frame for QR codes:

```python
decoded_objects = decode(frame)
```

Once a QR code is detected, its corner coordinates are extracted.

These four corners provide the reference points required for pose estimation.

---

### Step 3 — Define the QR Code in 3D

The physical QR code is treated as a square planar object with known dimensions.

For example:

```python
qr_size = 0.5
```

Its four corners are represented in 3D space as:

```text
(0, 0, 0)
(qr_size, 0, 0)
(qr_size, qr_size, 0)
(0, qr_size, 0)
```

This establishes the QR code as the coordinate system for the augmented object.

---

### Step 4 — Estimate Camera Pose

OpenCV's Perspective-n-Point algorithm is used to determine the QR code's orientation and position relative to the camera.

```python
cv2.solvePnP()
```

The algorithm produces:

* **Rotation vector (`rvec`)**
* **Translation vector (`tvec`)**

Together, these describe the pose of the QR code relative to the camera.

---

### Step 5 — Project the Virtual Object

The virtual object's 3D coordinates are defined in the QR code's coordinate system.

For example, the cube contains eight vertices:

```text
Bottom:
(0,0,0)
(1,0,0)
(1,1,0)
(0,1,0)

Top:
(0,0,1)
(1,0,1)
(1,1,1)
(0,1,1)
```

These points are transformed from 3D coordinates into 2D image coordinates using:

```python
cv2.projectPoints()
```

The resulting points are then connected with OpenCV drawing functions to create the virtual object.

---

## 3D Cube

The cube module demonstrates **marker-based augmented reality**.

The QR code provides the planar reference while the cube exists in a virtual 3D coordinate system.

The cube's:

* Position
* Orientation
* Perspective

change automatically as the QR code moves relative to the camera.

---

## 3D Pyramid

The pyramid module uses the same pose-estimation process but changes the virtual geometry.

The pyramid consists of:

* Three base vertices
* One apex

The apex is positioned above the QR-code plane, allowing the pyramid to appear as a 3D object anchored to the marker.

---

## Video Augmentation

The video module demonstrates a different type of augmented reality.

Instead of calculating a 3D object, the detected QR code is used as a **visual trigger**.

Once detected:

1. The QR code's bounding rectangle is calculated.
2. A frame is read from the video.
3. The video frame is resized to the QR code dimensions.
4. The resized frame is placed over the QR code region.
5. The resulting frame is displayed through the PyQt5 interface.

This creates a simple marker-based AR content-display system.

---

## Installation

### 1. Clone the repository

```bash
git clone <repository-url>
cd QR-AR
```

### 2. Install dependencies

```bash
pip install opencv-python numpy pyzbar PyQt5
```

You can also install them from a requirements file:

```bash
pip install -r requirements.txt
```

Example `requirements.txt`:

```text
opencv-python
numpy
pyzbar
PyQt5
```

> **Note:** PyZBar may require the ZBar library depending on the operating system.

---

## Running the Project

### Cube

```bash
python cube.py
```

### Pyramid

```bash
python pyramid.py
```

### Video Overlay

```bash
python video_overlay.py
```

Make sure a webcam is connected and accessible by OpenCV.

For the video-overlay module, place the video file in the project's video directory and update the path in the script if necessary.

---

## Requirements

* Python 3.x
* Webcam
* Printed or displayed QR code
* OpenCV
* NumPy
* PyZBar
* PyQt5
* ZBar runtime/library where required
* Video file for the video-overlay module

---

## Important Implementation Notes

### Camera Calibration

The current implementation creates an approximate camera matrix based on the frame dimensions:

```python
camera_matrix = np.array([
    [frame.shape[1], 0, frame.shape[1] / 2],
    [0, frame.shape[1], frame.shape[0] / 2],
    [0, 0, 1]
], dtype=np.float32)
```

This is sufficient for demonstrating the concept, but it is **not a substitute for proper camera calibration**.

For a more accurate AR system, the camera should be calibrated using a calibration pattern such as a checkerboard to obtain:

* Focal lengths
* Principal point
* Lens distortion coefficients

---

## Limitations

The current implementation is designed as a demonstration/prototype and has several limitations:

* Camera parameters are approximated rather than calibrated.
* QR code size is assumed rather than measured automatically.
* The video overlay uses a rectangular bounding box rather than perspective warping.
* Detection may become unstable when the QR code is heavily occluded or viewed at extreme angles.
* Lighting and camera quality can affect QR detection.
* The video overlay currently resets when the video reaches the end.
* Multiple QR codes are not treated as independent AR targets.

---

## Future Improvements

Possible extensions include:

* [ ] Proper camera calibration
* [ ] Perspective-correct video projection
* [ ] Multiple QR-code tracking
* [ ] Different 3D models and animations
* [ ] Interactive AR objects
* [ ] QR-code-specific content mapping
* [ ] Improved pose stabilization
* [ ] Real-time object tracking
* [ ] Texture mapping
* [ ] Support for custom 3D models
* [ ] Improved GUI controls
* [ ] Mobile/webcam deployment
* [ ] Integration with a 3D rendering engine

---

## Applications

The project demonstrates concepts that can be applied to:

* Augmented reality
* Interactive learning
* Product visualization
* Digital marketing
* Museum and exhibition experiences
* Interactive packaging
* QR-based information systems
* Educational demonstrations
* Marker-based robotics and computer vision

---

## Core Computer Vision Concepts

This project provides practical implementation of several important computer vision concepts:

**QR Detection**

Identifying and extracting the location of a QR code from an image.

**Feature/Corner Localization**

Using the QR code's four corners as reference points.

**Camera Pose Estimation**

Estimating the 3D position and orientation of a planar object relative to the camera.

**Perspective Projection**

Mapping 3D coordinates onto the 2D camera image.

**Augmented Reality**

Combining real-world camera frames with digitally generated content.

**Real-Time Processing**

Continuously processing webcam frames and updating the augmented output.

---

## Project Objective

The primary objective of this project is to demonstrate a **marker-based augmented reality system using QR codes**.

By combining QR detection, 3D geometry, pose estimation and real-time image processing, the project transforms a conventional QR code into an interactive AR marker capable of displaying virtual 3D objects and multimedia content.

---

## Author

**[Your Name]**

Developed as a computer vision / augmented reality project using Python and OpenCV.

