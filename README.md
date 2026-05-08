# Reflection and Glare Noise Removal for Autonomous Vehicle Vision

## Overview

This repository focuses on **road reflection and glare noise removal** for autonomous vehicle vision systems.
The goal of this project is to improve the reliability of lane detection and navigation algorithms in environments where strong light reflections, glare, or high-contrast regions appear on the road surface.

In autonomous driving systems, reflective surfaces can easily be misinterpreted as lane markings or road boundaries. This causes incorrect line detection and leads to unstable steering behavior and deviation from the center of the road.

This project acts as a **preprocessing stage** before the lane detection and navigation pipeline.

The navigation and line-following system itself is implemented in a separate repository.
This repository is specifically dedicated to **visual noise reduction and reflection handling**.

---

# Problem

Road reflections caused by:

* Sunlight
* Wet roads
* High brightness surfaces
* Camera overexposure
* Artificial lights

can introduce false edges and false lane-like patterns.

Traditional edge detection algorithms such as:

* Canny Edge Detection
* Hough Line Transform

may incorrectly detect these reflections as valid road lines.

As a result:

* The estimated lane center becomes inaccurate
* Steering control becomes unstable
* The autonomous car may drift away from the road center

---

# Solution

The proposed solution removes high-contrast reflective regions before the lane detection stage.

The pipeline includes:

1. Brightness reduction
2. High contrast detection
3. Reflection masking
4. Noise black-out
5. Image inpainting
6. Clean image generation for lane detection

The cleaned image is then passed into the navigation system.

---

# Processing Pipeline

## 1. Local Brightness Reduction

The image brightness is normalized using HSV color space processing and gamma correction.

This helps reduce the intensity of extremely bright areas while preserving useful road information.

```python
frame = reduce_local_brightness(frame,
                                threshold=1,
                                gamma=1.5,
                                kernel_size=10)
```

### Purpose

* Reduce local overexposure
* Prevent extremely bright reflections
* Improve contrast consistency

---

## 2. High Contrast Detection

The system detects sharp intensity changes using the Laplacian operator.

```python
laplacian = cv2.Laplacian(gray, cv2.CV_64F)
```

High contrast areas usually correspond to:

* Glare
* Reflections
* Sharp light artifacts

These regions are extracted using thresholding.

```python
_, high_contrast_mask = cv2.threshold(
    laplacian_abs,
    40,
    255,
    cv2.THRESH_BINARY
)
```

---

## 3. Morphological Cleanup

Dilation and erosion are applied to clean the reflection mask.

```python
high_contrast_mask = cv2.dilate(...)
high_contrast_mask = cv2.erode(...)
```

### Purpose

* Remove small noise artifacts
* Connect fragmented reflection regions
* Produce smoother masks

---

## 4. Reflection Blackout

Detected glare regions are temporarily removed by converting them to black.

```python
image_masked[high_contrast_mask == 255] = [0, 0, 0]
```

This prevents the line detection system from considering these regions as road lanes.

---

## 5. Image Inpainting

The removed regions are reconstructed using OpenCV inpainting.

```python
cv2.inpaint(...)
```

### Purpose

* Restore road continuity
* Preserve surrounding road texture
* Generate smoother input for edge detection

---

# Integration with Navigation System

After preprocessing, the cleaned frame is sent to the lane detection pipeline.

The lane detection system uses:

* Gaussian Blur
* Canny Edge Detection
* Hough Line Transform
* Lane averaging
* PID steering control

Without reflection removal, false lane detections significantly affect steering calculations.

By cleaning the image beforehand, the navigation system becomes:

* More stable
* More accurate
* Less sensitive to light noise
* Better centered on the road

---

# Main Components

## Reflection Removal

```python
def black_noise(image):
```

Handles:

* Reflection detection
* Contrast masking
* Inpainting

---

## Brightness Reduction

```python
def reduce_local_brightness(image,
                            threshold=1,
                            gamma=1.0,
                            kernel_size=10):
```

Handles:

* Brightness normalization
* Gamma correction
* Local illumination balancing

---

## Lane Detection

```python
def Line_Detection(img):
```

Handles:

* Edge extraction
* Hough line detection
* Lane averaging
* Road center estimation
* Steering angle generation

---

# Technologies Used

* Python
* OpenCV
* NumPy
* Raspberry Pi
* PiCamera2
* Arduino Serial Communication

---

# Hardware Setup

* Raspberry Pi 5
* Pi Camera
* Arduino
* Servo Motor
* Ackermann Steering Platform

---

# Output

The system generates:

* Reflection masks
* Blacked-out glare regions
* Reconstructed clean frames
* Improved lane detection stability
* More reliable steering control

---

# Example Workflow

```text
Camera Frame
     ↓
Brightness Reduction
     ↓
Reflection Detection
     ↓
Mask Cleanup
     ↓
Reflection Blackout
     ↓
Inpainting Reconstruction
     ↓
Lane Detection
     ↓
PID Steering Control
     ↓
Vehicle Navigation
```

---

# Repository Purpose

This repository is intended to demonstrate:

* Computer vision preprocessing for autonomous vehicles
* Reflection and glare handling techniques
* Real-time image enhancement
* Robust lane detection preparation
* Embedded autonomous driving research

---

# Future Improvements

Possible future enhancements include:

* Adaptive reflection thresholding
* Deep learning based glare segmentation
* Temporal filtering between frames
* HDR camera integration
* Reflection-aware lane detection models

---

# Related Repository

The lane detection and autonomous navigation system is available in a separate repository.

This repository only focuses on:

* Reflection removal
* Light noise reduction
* Image preprocessing for navigation systems

---

# Author

Fatemeh Mirashrafi

Computer Vision • Autonomous Systems • Robotics • Embedded AI
