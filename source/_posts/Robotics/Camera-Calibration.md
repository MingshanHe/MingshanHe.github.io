---
title: Camera_Calibration
date: 2022-04-05 22:12:27
tags: camera-calibration
categories: Robotics
math: true
---
# Camera Calibration

## Goal

* types of distortion caused by cameras
* how to find the intrinsic and extrinsic properties of a camera
* how to undistort images based off these properties

## Basics

  Some pinhole cameras introduce significant distortion to images. Two major kinds of distortion are radial distortion and tangetial distortion.

* Radial distortion causes straight lines to appear curved. Radial distortion becomes larger the farther points are from the center of the image. Radial distortion can be represented as follows:
  $$
  x_{distorted}=x(1+k_1r^2+k_2r^4+k_3r^6)\\
  y_{distorted}=y(1+k_1r^2+k_2r^4+k_3r^6)
  $$

* 

* Tangetial distortion occurs because the image-taking lense is **not aligned perfectly parallel** to the imaging plane.
  $$
  x_{distorted}=x+[2p_1xy+p_2(r^2+2x^2)]\\
  y_{distorted}=y+[p_1(r^2+2y^2)+2p_2xy]
  $$

  It's need to find five parameters, known as distortion coefficients give by ***Distortion coefficients***:$k_1$  $k_2$  $p_1$  $p_2$  $k_3$ .

  In addition to this, it need to some other information, like the intrinsic and extrinsic parameters of the camera. ***Intrinsic parameters*** are specific to a camera. They include information like focal length $(f_x,f_y)$ and optical centers $(c_x, c_y)$. The focal length and optical centers can be used to create a camera matrix, which can be used to remove distortion due to the lenses of a specific camera. The camera matrix is unique to a specific camera, so once calculated, it can be reused on other images taken by the same camera. The matrix is represented as:

$$
camera\space matrix=\begin{bmatrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}
$$

  ***Extrinsic parameters*** corresponds to rotation and translation vectors which translates a coordinates of a 3D point to a coordinate system.

## More Details

  Let's simplify the process of potographing and the model of camera, which illustrated as follows,

<img src="/image/Robotics/Camera_Calibration/1.png" alt="3" style="zoom:100%;" />

<center>Figure 1. Simple Model of Camera</center>

  Firstly, the coordinate of camera has been established by the center of light as the original point $O$, and the direct of $X$ and $Y$ are the two directions of CCD pixel arrangement. The direct of $Z$ could be corrected by the right hand coordinate. Secondly, the coordinate of image established as two dimensions as the directions $U$ and $V$. 

<img src="/image/Robotics/Camera_Calibration/2.png" alt="3" style="zoom:100%;" />

<center>Figure 2. Mathmetic Model of Camera</center>

* From the center of light $O$, the plane of image at $Z = f $, where $f$ is the physical focal length (unit: mm).

* Point Q is in the space, which location is $Q(X,Y,Z)$ in the coordinate of camera.

* Point P is in the plane of image, and it has two same description, like position $(x,y,f)$ in the coordinate of image plane and position $(u_{ccd},v_{ccd})$ in coordinate of pixel arrangement.

* $k$ and $l$ are the length of two directions of one CCD pixel (unit: mm/pixel), so the definition of focus length are $f_x=\frac{f}{k}$ and $f_y=\frac{f}{l}$ (Unit: pixel).

* If the offsets from the original point of the coordinate to the light axial are $c_x$ and $c_y$ (unit: pixel)

* So the map between the position in pixel $(u_{ccd},v_{ccd})$ and the position in three dimensional space $(X, Y, Z)$ Is:
  $$
  \begin{bmatrix}u_{ccd}*Z\\v_{ccd}*Z\\Z\end{bmatrix}=\begin{bmatrix}f_x&0&c_x\\0&f_y&c_y\\0&0&1\end{bmatrix}\begin{bmatrix}X\\Y\\Z\end{bmatrix}
  $$

## The Purpose of Camera Calibration

  The purpose of the calibration process is to find the matrix $K\in\R^3$, the rotation matrix $R\in\R^3$, and the translation vector $t\in\R^3%=$ using a set of known 3D points $(X_w, Y_w, Z_w)$ and their corresponding image coordinates $(u,v)$. When we get the values of intrinsic and extrinsic parameters the camera is said to be calibrated.

  In summary, a camera calibration algorithms has the following inputs and outputs

1. **Inputs**: A collection of images with points whose 2D image coordinates and 3D world coordinates are known.
2. **Outputs**: The camera intrinsic matrix, the rotation and translation of each image.

##  Different Types of Camera Calibration Methods

* Calibration pattern, Geometric clues, Deep Learning based

## Camera Calibration Step by Step

<img src="/image/Robotics/Camera_Calibration/3.png" alt="3" style="zoom:65%;" />

<center>Figure 3. Overview of Camera Calibration</center>

### Step 1: Define real world coordinates with checkerboard pattern

  **World Coordinate System**: In the calibration, the world coordinates are fixed by this checkerboard pattern that is attached to a wall in the room. Any corner of the above board can be chosen to the origin of the world coordinate system. The $X_{w}$ and $Y_{w}$ axes are along the wall, and the $Z_{w}$ axis is perpendicular to the wall. All points on the checkerboard are therefore on the XY plane (i.e. $Z_{w}=0$).

  In the process of calibration we calculate the camera parameters by a set of know 3D points $(X_{w}, Y_{w}, Z_{w})$ and their corresponding pixel location $(u,v)$ in the image.

  For the 3D points we photograph a checkerboard pattern with known dimensions at many different orientations. The world coordinate is attached to the checkerboard and since all the corner points lie on plane, we can arbitrarily choose $Z_{w}$ for every point to be 0.

  #### Why is the ckeckerboard pattern so widely used in calibration?

  Checkerboard patterns are distinct and easy to detect in an image. Not only that, the corners of squares on the checkerboard are ideal for localizing them because they have sharp gradients in two directions.

### Step 2: Capture multiple images of the checkerboard from different viewpoints

  Next, we keep the checkerboard static and take multiple images of the checkerboard by moving the camera. Alternatively, we can also keep the camera constant and photograph the checkerboard pattern at different orientations.

### Step 3: Find 2D coordinates of checkerboard

  We now have multiple of images of the checkerboard. We also know the 3D location of points on the checkerboard in world coordinates. The last thing we need are the 2D pixel locations of these checkerboard corners in the images.

#### 3.1 Find checkerboard corners

  OpenCV provides a builtin function called `findChessboardCorners` that looks for a checkerboard and returns the coordinates of the corners. Its usage is given by

**C++**

```cpp
bool findChessboardCorners (InputArray iamge, Size patternSize, OutputArray corners, int flags = CALIB_CB_ADAPTIVE_THRESH + CALIB_CB_NORMALIZE_IMAGE)
```

**Python**

```python
retval, corners = cv2.findCHessboardCorners (image, patternSize, flags)
```

Where, `image` is the source chessboard view (an 8-bit grayscale or color image), `patternSize` is the number of inner corners per a chessboard row and column ( `patternSize = cvSize (points_per_row, points_per_colum) = cvSize(columns, rows)`), `corners` is the output array of detected corners, and flags are the various operation flags.

#### 3.2 Refine checkerboard corners

  Good calibration is all about precision. To get good results, it is important to obtain the location of corners with sub-pixel level of accuracy. The function `cornerSubPix` takes in the original images, and the location of corners, and looks for the best corner location inside a small neighborhood of the original location.

**C++**

```cpp
void cornerSubPix (InputArray image, InputOutputArray corners, Size winSize, Size zeroZone, TermCriteria criteria)
```

**Python**

```python
cv2.cornerSubPix (image, corners, winSize, zeroZone, criteria)
```

Where, `image` is the imput image, `corners` are the initial coordinates of the input corners and refined coordinates provided for output, `winSize` is the half of the side length of the search window, `zeroZone` is the half of the size of the dead region in the middle of the search zone owver which the summation in the formula below is not done, and `criteria` is for the termination of the iterative process of corner refinement.

### Step 4: Calibrate Camera

The final step of calibration is to pass the 3D points in world coordinates and their 2D locations in all images to `calibrateCamera` method.

**C++**

```cpp
double calibrateCamera (InputArrayOfArrays objectPoints, InputArrayOfArrays iamgePoints, Size imageSize, InputOutputArray cameraMatrix, InputOutputArray distCoeffs, OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs)
```

**Python**

```python
retval, cameraMatrix, distCoeffs, rvecs, tvecs = cv2.calibrateCamera (objectPoints, imagePionts, imageSize)
```

Where, `objectPoints` is a vector of vectors of 3D points, `ImagePoints` is a vector of vectors of the 2D image points, `ImageSize` is the size of the image, `cameraMatrix` is the intrinsic camera matrix, `distCoeffs` is the lens distortion coefficients, `rvecs` is the rotation specified as a 3$\times$1 vector, and `tvecs` is the 3$\times$1 vector.

## Camera Calibration Code

### **C++ Code**

```cpp
#include <opencv2/opencv.hpp>
#include <opencv2/calib3d/calib3d.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>
#include <stdio.h>
#include <iostream>
 
// Defining the dimensions of checkerboard
int CHECKERBOARD[2]{6,9};

int main()
{
  // Creating vector to store vectors of 3D points for each checkerboard image
  std::vector<std::vector<cv::Point3f>> objpoints;
 
  // Creating vector to store vectors of 2D points for each checkerboard image
  std::vector<std::vector<cv::Point2f>> imgpoints;
 
  // Defining the world coordinates for 3D points
  std::vector<cv::Point3f> objp;

  for(int i{0}; i<CHECKERBOARD[1]; i++)
  {
    for(int j{0}; j<CHECKERBOARD[0]; j++)
      objp.push_back(cv::Point3f(j,i,0));
  }
 
  // Extracting path of individual image stored in a given directory
  std::vector<cv::String> images;
  
  // Path of the folder containing checkerboard images
  std::string path = "./images/*.jpg";
 
  cv::glob(path, images);

  cv::Mat frame, gray;

  // vector to store the pixel coordinates of detected checker board corners
  std::vector<cv::Point2f> corner_pts;
  bool success;

  // Looping over all the images in the directory
  for(int i{0}; i<images.size(); i++)
  {
    frame = cv::imread(images[i]);
    cv::cvtColor(frame,gray,cv::COLOR_BGR2GRAY);
    // Finding checker board corners
    // If desired number of corners are found in the image then success = true 
    success = cv::findChessboardCorners(gray, cv::Size(CHECKERBOARD[0], CHECKERBOARD[1]), corner_pts, CV_CALIB_CB_ADAPTIVE_THRESH | CV_CALIB_CB_FAST_CHECK | CV_CALIB_CB_NORMALIZE_IMAGE);
    /*
     * If desired number of corner are detected,
     * we refine the pixel coordinates and display
     * them on the images of checker board
    */
    if(success)
    {
      cv::TermCriteria criteria(CV_TERMCRIT_EPS | CV_TERMCRIT_ITER, 30, 0.001);
      // refining pixel coordinates for given 2d points.
      cv::cornerSubPix(gray,corner_pts,cv::Size(11,11), cv::Size(-1,-1),criteria);
      
      // Displaying the detected corner points on the checker board
      cv::drawChessboardCorners(frame, cv::Size(CHECKERBOARD[0], CHECKERBOARD[1]), corner_pts, success);
       
      objpoints.push_back(objp);
      imgpoints.push_back(corner_pts);
    }
    cv::imshow("Image",frame);
    cv::waitKey(0);
  }
  cv::destroyAllWindows();
  cv::Mat cameraMatrix,distCoeffs,R,T;
  /*
   * Performing camera calibration by
   * passing the value of known 3D points (objpoints)
   * and corresponding pixel coordinates of the
   * detected corners (imgpoints)
  */
  cv::calibrateCamera(objpoints, imgpoints, cv::Size(gray.rows,gray.cols), cameraMatrix, distCoeffs, R, T);

  std::cout << "cameraMatrix : " << cameraMatrix << std::endl;
  std::cout << "distCoeffs : " << distCoeffs << std::endl;
  std::cout << "Rotation vector : " << R << std::endl;
  std::cout << "Translation vector : " << T << std::endl;

  return 0;
}
```

### **Python Code**

```python
#!/usr/bin/env python

import cv2
import numpy as np
import os
import glob

# Defining the dimensions of checkerboard
CHECKERBOARD = (6,9)
criteria = (cv2.TERM_CRITERIA_EPS + cv2.TERM_CRITERIA_MAX_ITER, 30, 0.001)

# Creating vector to store vectors of 3D points for each checkerboard image
objpoints = []
# Creating vector to store vectors of 2D points for each checkerboard image
imgpoints = [] 


# Defining the world coordinates for 3D points
objp = np.zeros((1, CHECKERBOARD[0] * CHECKERBOARD[1], 3), np.float32)
objp[0,:,:2] = np.mgrid[0:CHECKERBOARD[0], 0:CHECKERBOARD[1]].T.reshape(-1, 2)
prev_img_shape = None

# Extracting path of individual image stored in a given directory
images = glob.glob('./images/*.jpg')
for fname in images:
    img = cv2.imread(fname)
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    # Find the chess board corners
    # If desired number of corners are found in the image then ret = true
    ret, corners = cv2.findChessboardCorners(gray, CHECKERBOARD, cv2.CALIB_CB_ADAPTIVE_THRESH + cv2.CALIB_CB_FAST_CHECK + cv2.CALIB_CB_NORMALIZE_IMAGE)
    
    """
    If desired number of corner are detected,
    we refine the pixel coordinates and display 
    them on the images of checker board
    """
    if ret == True:
        objpoints.append(objp)
        # refining pixel coordinates for given 2d points.
        corners2 = cv2.cornerSubPix(gray, corners, (11,11),(-1,-1), criteria)
        
        imgpoints.append(corners2)

        # Draw and display the corners
        img = cv2.drawChessboardCorners(img, CHECKERBOARD, corners2, ret)
    
    cv2.imshow('img',img)
    cv2.waitKey(0)

cv2.destroyAllWindows()

h,w = img.shape[:2]

"""
Performing camera calibration by 
passing the value of known 3D points (objpoints)
and corresponding pixel coordinates of the 
detected corners (imgpoints)
"""
ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None)

print("Camera matrix : \n")
print(mtx)
print("dist : \n")
print(dist)
print("rvecs : \n")
print(rvecs)
print("tvecs : \n")
print(tvecs)
```

