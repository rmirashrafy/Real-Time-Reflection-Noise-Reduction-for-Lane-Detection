

# Real-Time Reflection Noise Reduction for Lane Detection

## Overview

This repository focuses on road reflection and glare noise removal for autonomous vehicle vision systems. The goal is to improve the reliability of lane detection and navigation algorithms in environments with strong sunlight reflections, glare, and high-contrast road surfaces.

In autonomous driving systems, reflective surfaces can be misinterpreted as lane markings or road boundaries, which leads to unstable steering and incorrect path estimation. This project serves as a **preprocessing stage** before lane detection and navigation.

The actual navigation and line-following system is implemented in a separate repository. This project is dedicated purely to **visual noise reduction and reflection handling**.



## Problem

Road reflections caused by sunlight, wet surfaces, high-brightness materials, camera overexposure, and artificial lighting introduce false edges and lane-like artifacts in the perception pipeline.

Traditional methods such as Canny Edge Detection and Hough Line Transform may incorrectly interpret these reflections as valid lane markings. This results in inaccurate lane estimation, unstable steering, and vehicle drift from the intended trajectory.

<img width="500" height="350" alt="6037193665950106864" src="https://github.com/user-attachments/assets/b6ed8168-bb15-45cc-91a7-2b76d96dd7da" />


## Solution

The proposed approach suppresses high-contrast reflective regions before lane detection to improve perception stability under challenging lighting conditions.

The pipeline performs brightness normalization, reflection detection, mask refinement, glare suppression, and image reconstruction using inpainting. This produces a cleaner frame for downstream lane detection and autonomous navigation.



## Processing Pipeline

Each frame is processed through a real-time enhancement pipeline designed to remove glare and reflection artifacts.

First, local brightness normalization is applied using HSV-based processing and gamma correction to reduce overexposed regions while preserving road structure:

```python
frame = reduce_local_brightness(
    frame,
    threshold=1,
    gamma=1.5,
    kernel_size=10
)
````

To detect reflective and high-contrast regions, the Laplacian operator is used:

```python
laplacian = cv2.Laplacian(gray, cv2.CV_64F)
```

A binary mask is then created using thresholding:

```python
_, high_contrast_mask = cv2.threshold(
    laplacian_abs,
    40,
    255,
    cv2.THRESH_BINARY
)
```

Morphological operations (dilation and erosion) are applied to clean and refine the mask by removing noise and connecting fragmented regions.

Detected reflection areas are then suppressed by blacking them out:

```python
image_masked[high_contrast_mask == 255] = [0, 0, 0]
```

Finally, missing regions are reconstructed using OpenCV inpainting to restore road continuity:

```python
cv2.inpaint(...)
```

This results in a cleaner frame with reduced glare interference for downstream processing.



## Integration with Navigation System

After preprocessing, the cleaned image is passed to the lane detection and autonomous navigation pipeline.

The navigation system uses Gaussian blur, Canny edge detection, Hough line transform, lane averaging, and PID-based steering control to estimate the road center and generate steering commands.

Without reflection suppression, glare can create false lane detections and reduce steering stability. By removing reflective noise before perception, the system becomes significantly more robust in real-world outdoor lighting conditions.

The implementation runs on a Raspberry Pi–based robotic platform using Python, OpenCV, and NumPy, integrated with PiCamera2, Arduino serial communication, and servo-based Ackermann steering control.

<table>
  <tr>
    <td><img width="350" height="225" src="https://github.com/user-attachments/assets/952130b9-9e66-491b-b381-4a56cde3174f" /></td>
    <td><img width="350" height="225" src="https://github.com/user-attachments/assets/b395eaec-2717-481a-9b40-065e6be49f2a" /></td>
  </tr>
</table>


## Summary

This repository implements a real-time computer vision preprocessing module for autonomous driving systems. It focuses on reflection suppression, brightness normalization, and image restoration to improve lane detection robustness under challenging lighting conditions.

The system processes each camera frame through brightness correction, reflection detection, mask refinement, glare suppression, and inpainting before lane detection and PID steering control.



## Future Work

Possible improvements include adaptive thresholding, temporal filtering across frames, HDR-based preprocessing, deep learning–based reflection segmentation, and reflection-aware lane detection models.



