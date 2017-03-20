# Advanced Lane Finding Project

### - Mukil Kesavan

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

[ccal1]: ./output_images/camera_cal/undist.png "Undistorting Images"
[test1]: ./test_images/test1.jpg "Road Transformed"
[ccal2]: ./output_images/camera_cal/undist-test1.png "Undistorted Test Image"

[thresh1]: ./output_images/thresholding/orig.png "Undistorted Test Image"
[thresh2]: ./output_images/thresholding/schannel.png "S-Channel Color Thresholded Image"
[thresh3]: ./output_images/thresholding/gradabs.png "GradientX Direction"
[thresh4]: ./output_images/thresholding/gradmag.png "Gradient Magnitude"
[thresh5]: ./output_images/thresholding/graddir.png "Gradient Direction"
[thresh6]: ./output_images/thresholding/combined.png "Combined Binary Image"

[pt1]: ./output_images/perspective_transform/slimg2.png "Original Straight Lines Image"
[pt2]: ./output_images/perspective_transform/slimg2-warped.png "Warped Straight Lines Image"

[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./solution.ipynb" (code cell 3) under the title "Camera Calibration". I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the real world. I assume that the chessboard is placed on a 2-d plane with z=0 for all points. I then read in all the images from the "camera_cal" folder, convert them to grayscale and find the chessboard corners in each image using OpenCV findChessboardCorners() function, and append to "image points". "Image points" contains the x,y pixel positions of all the detected corners in the calibration images. I finally feed in the entire set of "image points" found in the calibration images and the hand crafted "object points" to OpenCV's calibrateCamera function to get the camera matrix and distortion coefficients. Note that the "object points" do not change for the calibration images.

I then used the camera matrix and distortion coefficients to undistort the calibration images using OpenCV's undistort function. Here's an example:

![alt text][ccal1]

### Pipeline (single test images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][test1]

In the code cell that defined the pipeline() function that contains the overall lane detection pipeline, the very first step is to undistort the input image. When this is applied to the above test image, we get the following result:

![alt text][ccal2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code to perform color and gradient thresholding is contained the in the IPython notebook located in "./solution.ipynb" in code cell under the title "Gradient and Color Thresholding". The overall pipeline() function first converts the image to HLS colorspace and uses S-Channel thresholding and Gradient thresholding in the X-direction to produce the binary image. This is in the code cell titled "Overall Pipeline" - please pay attention to combined_image calculation. I tried various other thresholding combinations and this was the one that worked best for me - the others were either too noisy or did not detect the lanes properly. The output of this can be seen below for a test image:

![alt text][thresh1] ![alt text][thresh2] ![alt text][thresh3] ![alt text][thresh4] ![alt text][thresh5] ![alt text][thresh6]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code to do perspective transformation is in IPython notebook located in "./solution.ipynb" in code cell titled "Perspective Transformation". The code to obtain the perspective transformation matrix and its inverse is in the getPerspectiveTransformationMatrix() function. I manually found the source  points for the test_images/straight_lines2.jpg image corresponding to the left and right lane lines at the bottom and top of the image.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 337, 633      | 200, 720      | 
| 975, 633      | 1000, 720     |
| 709, 463      | 1000, 0       |
| 575, 463      | 200, 0        |

I verified that my perspective transform was working as expected by drawing the polygon formed by the `src` and `dst` points onto the previously mentioned straight lines test image and its warped counterpart to verify that the lines appear parallel in the warped image. Here's the output:

![alt text][pt1] ![alt text][pt2]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

