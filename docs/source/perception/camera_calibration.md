<!-- Template Type: Process -->
<!-- Topic: Camera Calibration -->
# Camera Calibration

## Overview

Camera calibration is the process of estimating the parameters of a camera to correct distortions and achieve accurate measurements. It is crucial for applications in computer vision, robotics, and photogrammetry.

## Types of Calibration

## 1. Geometric Calibration

Geometric calibration involves determining the geometric properties of the camera, such as its position, orientation, and lens distortion. This is essential for applications that require precise spatial measurements.

### 1.1 Intrinsic Calibration

Intrinsic calibration focuses on the internal parameters of the camera, which include:

- **Focal Length**: The distance between the camera lens and the image sensor.
- **Principal Point**: The point on the image sensor where the optical axis intersects.
- **Skew Coefficient**: The angle between the x and y pixel axes.
- **Distortion Coefficients**: Parameters that describe the lens distortion, such as radial and tangential distortion.
**Goal:** Determine the camera matrix and lens distortion coefficients.

**Parameters to Estimate**:
Focal Length (fx, fy)
Optical Center (cx, cy)
Radial and Tangential Distortion (k1, k2, k3, p1, p2)

OpenCV Calibration Workflow:
**Code**

```python
import cv2
import numpy as np
import glob

# Define checkerboard dimensions
checkerboard_size = (9, 6)  # 9x6 inner corners

# Arrays to store object points and image points
objp = np.zeros((checkerboard_size[0]*checkerboard_size[1], 3), np.float32)
objp[:, :2] = np.mgrid[0:9, 0:6].T.reshape(-1, 2)  # 3D points in real-world space

objpoints = []  # 3D points in real world
imgpoints = []  # 2D points in image plane

images = glob.glob('calibration_images/*.jpg')

for fname in images:
    img = cv2.imread(fname)
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

    # Find the chessboard corners
    ret, corners = cv2.findChessboardCorners(gray, checkerboard_size, None)

    if ret:
        objpoints.append(objp)
        imgpoints.append(corners)
        cv2.drawChessboardCorners(img, checkerboard_size, corners, ret)

# Calibrate the camera
ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None)

print("Camera Matrix:\n", mtx)
print("Distortion Coefficients:\n", dist)
```

OpenCV Functions Used

- `cv2.findChessboardCorners()` – Detects checkerboard corners.
- `cv2.cornerSubPix()` – Refines corner locations to subpixel accuracy.
- `cv2.calibrateCamera()` – Performs full calibration, returning intrinsic/- extrinsic parameters and distortion coefficients.
- `cv2.undistort()` – Corrects images for lens distortion.
- `cv2.projectPoints()` – Projects 3D points into the 2D image plane using calibration parameters.
**Result:**

Camera Matrix (mtx): Intrinsic parameters.
Distortion Coefficients (dist): Lens distortion correction.

### 1.2 Extrinsic Calibration

Extrinsic calibration determines the camera's position and orientation in the world coordinate system. This involves:

- **Rotation Matrix**: Describes the camera's orientation.
- **Translation Vector**: Describes the camera's position relative to the world coordinate system.
**Goal:** Align the camera with the robot’s coordinate system or other sensors.

**Steps:**
Capture images of the calibration target (checkerboard/ArUco) in the robot workspace.
Use SolvePnP to compute rotation (R) and translation (T):
**Code**

```python
# SolvePnP for extrinsic calibration
ret, rvec, tvec = cv2.solvePnP(objp, corners, mtx, dist)

# Convert rotation vector to rotation matrix
R, _ = cv2.Rodrigues(rvec)

print("Rotation Matrix:\n", R)
print("Translation Vector:\n", tvec)
```

Form the extrinsic matrix:
[𝑅∣𝑇] or as a 4x4 homogeneous transformation matrix

### 1.3 Multi-view Calibration

Multi-view calibration involves calibrating multiple cameras simultaneously to understand their relative positions and orientations. This is crucial for applications like 3D reconstruction and stereo vision.
**Stereo Camera Calibration (Optional for Depth Sensing)**
If using stereo cameras for depth estimation, calibrate both cameras jointly.
**Code**

```python
cv2.stereoCalibrate(objpoints, imgpoints_left, imgpoints_right,
                    mtx_left, dist_left, mtx_right, dist_right,
                    image_size, R, T, E, F)
```

Result:

R: Rotation between cameras.
T: Translation between cameras.
E: Essential matrix (used for 3D reconstruction).
F: Fundamental matrix (used for epipolar geometry).

## 2. Photometric Calibration

Photometric calibration deals with the camera's response to light and how it captures the intensity of the scene. This is important for applications that require accurate color and brightness measurements.

### 2.1 Radiometric Response Function

The radiometric response function describes the relationship between the scene radiance and the pixel values. It is used to correct the non-linear response of the camera sensor to light.
**Goal:** Linearize the relationship between scene brightness and pixel intensity. Map pixel values to actual scene irradiance.

Method: Capture images of a known radiometric target (e.g., grayscale chart) under controlled lighting.

Correction Formula:
**Method:** Capture a sequence of images with varying exposures and solve for the CRF.

**Code:**

```python
calibrate = cv2.createCalibrateDebevec()
merge_debevec = cv2.createMergeDebevec()

# Load images with different exposures
img_list = [cv2.imread(f"exposure_{i}.jpg") for i in range(1, 4)]
exposure_times = np.array([1/30.0, 1/60.0, 1/125.0], dtype=np.float32)

# Estimate camera response function
response_func = calibrate.process(img_list, exposure_times)

# Merge to create HDR image
hdr_image = merge_debevec.process(img_list, exposure_times, response_func)
```

### 2.2 Noise Level Estimation

Noise level estimation involves determining the amount of noise in the captured images. This is important for improving the quality of the images and for applications that require precise measurements.
Key Factors to Correct:
Sensor Noise: Readout noise, dark current.
Pixel Non-uniformity: Variation in sensitivity across pixels.
Lens Transmission Losses: Light absorption by the lens.

**Dark Frame Subtraction (Sensor Noise Correction)**
**Goal:** Remove fixed-pattern noise and thermal noise.

**Method:** Capture a dark frame (lens cap on) with the same exposure time.

**Code:**

```python
# Load dark frame
dark_frame = cv2.imread("dark_frame.jpg", cv2.IMREAD_GRAYSCALE)

# Subtract dark frame from raw image
corrected_image = cv2.subtract(raw_image, dark_frame)
```

2.3 Multi-Spectral and Thermal Camera Calibration
For thermal and hyperspectral cameras:

Blackbody Radiator: Used for thermal camera calibration.
Standard Light Sources (D65, A): Used for color calibration of multispectral sensors.

### 2.3 Vignetting

Vignetting refers to the reduction in image brightness or saturation at the periphery compared to the image center. Calibration helps in correcting this effect to achieve uniform brightness across the image.

**Method:** Capture images of a uniformly lit surface (e.g., a white wall or integrating sphere) to model how brightness falls off toward the image edges.

**Code:**

```python
import cv2
import numpy as np

# Load flat-field image (uniform white surface)
flat_field = cv2.imread("flat_field.jpg", cv2.IMREAD_GRAYSCALE)
vignetting_map = flat_field / np.max(flat_field)

# Load distorted image
raw_image = cv2.imread("raw_image.jpg", cv2.IMREAD_GRAYSCALE)

# Correct vignetting
corrected_image = raw_image / (vignetting_map + 1e-6)
corrected_image = cv2.normalize(corrected_image, None, 0, 255, cv2.NORM_MINMAX).astype(np.uint8)
```

### 2.4 Optical Blur (Spatial Response) Estimation

Optical blur estimation involves determining the amount of blur in the captured images due to the camera lens. This is important for applications that require sharp and clear images.

## Inputs

- **Calibration Patterns**: Images of known patterns (e.g., checkerboards) used for calibration.
- **Vanishing Points**: Points where parallel lines appear to converge in the image.
- **Radiometric Response Function**: The relationship between the scene radiance and the pixel values.

## Outputs

- **Intrinsic Parameters**: Camera-specific parameters such as focal length, principal point, and skew.
- **Extrinsic Parameters**: Parameters that describe the camera's position and orientation in the world.
- **Distortion Coefficients**: Parameters that describe lens distortion.

## Steps

1. **Data Collection**: Capture multiple images of a calibration pattern from different angles.
2. **Feature Extraction**: Detect and extract features (e.g., corners of a checkerboard) from the images.
3. **Parameter Estimation**: Use algorithms to estimate the intrinsic and extrinsic parameters and distortion coefficients.

## Key Considerations

- **Accuracy**: Ensure high precision in feature detection and parameter estimation.
- **Robustness**: Handle noise and outliers in the calibration data.
- **Consistency**: Maintain consistent lighting and pattern visibility during data collection.

## Applications

- Robotics navigation
- Augmented reality
- 3D reconstruction

## Components

- Calibration patterns
- Camera calibration software
- Image processing tools

## Algorithms

- Linear and nonlinear optimization techniques
- Homogeneous transformation approach
- Pose estimation algorithms

## Best Practices and Pitfalls

- Use high-quality calibration patterns.
- Avoid reflections and shadows in calibration images.
- Validate calibration results with test images.

## Interview Questions

- What are intrinsic and extrinsic parameters in camera calibration?
- How do you handle lens distortion in camera calibration?
- Explain the difference between linear and nonlinear camera calibration methods.

## Resources

### Trends

- Advances in real-time camera calibration
- Use of machine learning for improved calibration accuracy

### Tools and Techniques Used

- *MATLAB* Camera Calibration App
- *Calib.io:* For precise, factory-level calibration.
- *Basler pylon SDK, FLIR Spinnaker SDK:* Sensor-specific calibration tools.
- *OpenCV:* Open-source, customizable calibration framework.
- *Kalibr:* Advanced tool for multi-sensor calibration (IMU, camera, LiDAR).
- *AprilCal:* AprilTag-based calibration for high accuracy.
- *ChArUco in OpenCV:* Hybrid calibration for better feature detection.

### Courses

- Computer Vision courses on Coursera and edX
- Robotics courses focusing on perception

### Books

- "Multiple View Geometry in Computer Vision" by Richard Hartley and Andrew Zisserman
- "Computer Vision: Algorithms and Applications" by Richard Szeliski

### Blogs

- OpenCV blog
- Robotics and computer vision research blogs

### Video Tutorials

- YouTube tutorials on OpenCV camera calibration
- Online lectures from university courses

### Code Repo

- OpenCV GitHub repository
- MATLAB File Exchange for camera calibration tools

### Papers

- Research papers on camera calibration techniques and advancements

### Important Contributorsn

- Richard Hartley
- Andrew Zisserman
- Richard Szeliski
- Jean-Yves Bouguet
