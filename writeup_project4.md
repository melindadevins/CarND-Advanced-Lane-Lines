# CarND-Advanced-Lane-Lines Project 4

## Prerequisites

To run this project, you need [Miniconda](https://conda.io/miniconda.html) installed(please visit [this link](https://conda.io/docs/install/quick.html) for quick installation instructions.)

## Installation
To create an environment for this project use the following command:

```
conda env create -f environment.yml
```

After the environment is created, it needs to be activated with the command:

```
source activate carnd-term1-p4
```
---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[imageCorner]: ./output_images/calibration/cornerscalibration3.jpg "Corner in calibration3"
[imageOriginal]: ./output_images/calibration/calibration3orig.jpg "Origina Image calibration3"
[imageUndistorted]: ./output_images/calibration/calibration3undist.jpg "Undistorted calibration3"
[imageH]: ./output_images/transform/originalH.jpg "Undistorted original H"
[imageL]: ./output_images/transform/originalL.jpg "Undistorted original L"
[imageS]: ./output_images/transform/originalS.jpg "Undistorted original S"
[imageGradient]: ./output_images/transform/combineGradient.png "Combined Gradient"
[imageSource]: ./output_images/transform/perSource.jpg "Perspective Source"
[imageDest]: ./output_images/transform/perDestination.jpg "Perspective Destination"
[imagePers]: ./output_images/transform/perspectiveTransform.png "Perspective Transformation"
[imageLanePixel]: ./output_images/detect/lanePixel.png "Lane Pixel"
[imageLaneBoundary]: ./output_images/detect/laneBoundary.png "Lane Boundary"
[video1]: ./output_videos/project_video.mp4 "Video"

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for camera calibration is contained in the step 1 of the Jupyter notebook [Advance_Lane_Detection.ipynb].

The OpenCV functions `findChessboardCorners` and `calibrateCamera` are utilized to calibrate camera images.
A collection of chessboard images, taken from different angles with the same camera, are used as input images.
`findChessboardCorners` finds the internal corners location of the chessboard images in terms of pixel locations, which are arrays of object points.
Then the object points are fed to `calibrateCamera` which returns camera calibration and distortion coefficients.
These can then be used by the OpenCV `undistort` function to correct the effects of distortion on any image produced by the same camera.
 Generally, these coefficients will not change for a given camera (and lens).
 The below image is the corners drawn onto one of the chessboard images using the OpenCV function `drawChessboardCorners`:

![alt Undistorted image with corner][imageCorner]

### Pipeline (single images)

#### 2. Provide an example of a distortion-corrected image.
The following image shows the results of applying `undistort`, using the calibration and distortion coefficients,
to one of the chessboard images:

![alt Undistorted image with corner][imageOriginal] ![alt Undistorted image with corner][imageUndistorted]


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. Code is in step 3 of [Advance_Lane_Detection.ipynb]

First a color transformation to HLS was done. S channel was selected because it provides more contrast for the lane line,
as shown in the following images. The alst one is S channel.

![alt Undistorted original image H][imageH] ![alt Undistorted original image L][imageL] ![alt Undistorted original image S][imageS]

After the color transformation had been done, the combination of Sobel X and Sobel Y gradient was used.

The following image shows the binary image obtained with the combined gradient on S channel on the test images:

![alt Combinded gradient S channel][imageGradient]


#### 4. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in step 4 of [Advance_Lane_Detection.ipynb].
It takes the undistorted image of 'straight_lines1.jpg' as inputs, hard coded source (`src`) and destination (`dst`) points as the following:

Source:
[[  585.   455.]
 [  705.   455.]
 [ 1130.   720.]
 [  190.   720.]]

Destination:
[[  200.     0.]
 [ 1080.     0.]
 [ 1080.   720.]
 [  200.   720.]]

Use 'cv2.getPerspectiveTransform' to get the transformation matrix, and inverse transformation matrix as the following:

```python
M = cv2.getPerspectiveTransform(src, dst)
Minv = cv2.getPerspectiveTransform(dst, src)
```


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt Perspective Source][imageSource] ![alt Perspective Destination][imageDest]

The following picture shows the binary images results after the perspective transformation:

![alt Perspective Transformation][imagePers]

#### 5. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The line detection in step 5 of the [Advanced_Lane_Destection.ipynb] notebook.
First caculate the histogram on the X axis. Find the peak of the left and right halves of the histogram, as
the starting point for the left and right lines. Draw windows based on the mean position and average height.
Then collect the non-zero points contained on those windows. When all the points are collected,
a polynomial fit is used (using np.polyfit) to find the line.

The following picture shows the points found on each image, the windows and the polynomials:

![alt Lane Line Pixel][imageLanePixel]

#### 6. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

From step 5 a polynomial calculated on the meters space is used here to calculate the curvature.
The formula is the following:

```
((1 + (2*fit[0]*yRange*ym_per_pix + fit[1])**2)**1.5) / np.absolute(2*fit[0])
```

where `fit` is the the array containing the polynomial, `yRange` is the max Y value and `ym_per_pix`
is the meter per pixel value.

To find the vehicle position on the center:

- Calculate the lane center by evaluating the left and right polynomials at the maximum Y and find the middle point.
- Calculate the vehicle center transforming the center of the image from pixels to meters.

The code is in Step 6 of [Advanced_Lane_Destection.ipynb]

#### 7. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code is in Step 7 nd Step 8 of [Advanced_Lane_Destection.ipynb] A polygon is generated based on plots of the left and right fits, warped back to the perspective of the original image
using the inverse perspective matrix Minv and overlaid onto the original image. In addtion,
it also writes text identifying the curvature radius and vehicle position data onto the original image.
The image below is the result:

![alt Lane Area Identified][imageLaneBoundary]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There should be a better way to detect the lane than using histogram. Also, the way how the radius of the curvature of the lane seems complicated, and prune to error.

I think for self driving car, there should be markers on the road, and sensors and radar should be used to detect the markers thus the lane with certainty.
Computer vision should be utilized for object recognition, in addition to sensor and radar.

