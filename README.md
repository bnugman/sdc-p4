## Writeup Template

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

[corners]: ./output_images/corners1.jpg "With corners shown"
[undist]: ./output_images/calibration_undist1.jpg "Undistorted"
[test]: ./test_images/test3.jpg "Road Image Test 1"
[test_undist]: ./output_images/undist_test3.jpg "Road Image Test 1 Undistorted"

[test_magic]: ./output_images/magic_test3.jpg "Road Image Test 3 Channel mix"
[test_edgy]: ./output_images/edgy_test3.jpg "Road Image Test 3 Binarized"
[test_edgy_unwrap]: ./output_images/edgy_unwarp_test3.jpg "Road Image Test 3 Binarized, No persepctive"

[lane_fit]: ./output_images/lane_test3.jpg "Road Image Test 3 Lane fit"
[blended]: ./output_images/blended_test3.jpg "Road Image Test 3 Blended"


[de_perspect]: ./output_images/de_perspect_straight_lines1.jpg "Perspective removed"



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "LaneFinder.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

Distorted calibration image with corners identified:
![Distorted calibration image with corners][corners]

Undistorted calibration image:
![Undistorted][undist]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Original:
![original][test]

Undistorted:
![undistorted][test_undist]

The result may seem dubious, but seems to work. When I apply the same approach to the calibration images themselves, the result looks good, but it seems that the calibration images and the test images do not come from the same camera, or perhaps shot with different settings. In addition, some of the calibration images inexplicably have a different resolution, e.g. calibration7.jpg resolution is 1281x721, whereas previous images are in the 1280x720 resolution.

All the more reason to keep the pipeline modular and decoupled.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.

First, magic_channel() function combines saturation and red channels in a peculiar way (see code. If I were to be completely honest, saturation channel alone works about as well, but blending saturation and red channels seemed cooler.)

The resulting images show lanes prominently:

![channelized][test_magic]

I then apply Sobel transform in function edgify() with harcoded (but carefully chosen) thresholds of 30 and 200 to binarize the image.

The resulting binary image looks like this:

![binarized][test_edgy]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The functions related to perspective transforms are:

trapezoid() -- returns the trapezoid shape mimicking the perspective view of lanes derived from one of the undistorted images of straight lanes.

trapezoid_dst() -- returns a rectangular shape that we wish to "unwrap" the perspective view into

make_perspective_transform() -- returns the perspective transformation from trapezoid() shape into trapezoid_dst() shape

make_inv_perspective_transform() -- the inverse of the above

de_perspect() -- applies a perspective transformation

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![perspective removed for image of straight lanes][de_perspect]

And this is how the previously shown binarized image looks with perspective removed:

![binarized no perspective][test_edgy_unwrap]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial

The function find_and_fit_poly() searches for the lane curves in the following manner:

- build a histogram of the lower half of the image along the X axis
- identify the peaks of the histograms to the left and to the right of the midpoint; these are the "seeds" of our lane search
- use sliding windows to identify the nonzero pixels in the contigious area, adjust window position as the lanes shift
- finally, fit a polynomial of the specified degree to the pixels of the both lanes

Resulting fitted curves look like this:
![lane fit][lane_fit]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

curvature_and_shift_calc() calculates the radius of the curves, as well as the deviation of the midpoint between found lane curves and the midpoint of the picture.

Both the radius and the deviation from center are calculated in meters. One missing piece of the puzzle is how close the camera is mounted to the centerline of the vehicle. Judging from where the deviation values are centerd, the camera is mounted about 30cm to the left of the centerline, i.e. not far from where the drivers vantage point.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I apply inverse perspective tranform to the image of fitted lanes and then blend the resulting image with the original, like so:

![blended][blended]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.
