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

[ld1]: ./output_images/lane-detect/1.png "Sliding Window Lane Detection Histogram"
[ld2]: ./output_images/lane-detect/sliding1.png "Sliding Window Lane Detection"
[ld3]: ./output_images/lane-detect/fit1.png "Polynomial Lane Fit - Sliding Window"
[ld4]: ./output_images/lane-detect/errors.png "Fit Errors in Polynomial Coefficients"

[lf1]: ./output_images/lane-line-fit/laneplot.png "Lane Plot Result"

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

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code to perform color and gradient thresholding is contained the in the IPython notebook located in "./solution.ipynb" in code cell under the title "Gradient and Color Thresholding". The overall pipeline() function first converts the image to HLS colorspace and uses S-Channel thresholding and Gradient thresholding in the X-direction to produce the binary image. This is in the code cell titled "Overall Pipeline" - please pay attention to combined_image calculation. I tried various other thresholding combinations and this was the one that worked best for me - the others were either too noisy or did not detect the lanes properly. The output of this can be seen below for a test image:

![alt text][thresh1] ![alt text][thresh2] ![alt text][thresh3] ![alt text][thresh4] ![alt text][thresh5] ![alt text][thresh6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code to do perspective transformation is in IPython notebook located in "./solution.ipynb" in code cell titled "Perspective Transformation". The code to obtain the perspective transformation matrix and its inverse is in the getPerspectiveTransformationMatrix() function. Once I have the matrix, I simply call OpenCV's warpPerspective method to warp an image and get its top-down view. I manually found the source/destination  points for the test_images/straight_lines2.jpg image corresponding to the left and right lane lines at the bottom and top of the image.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 337, 633      | 200, 720      | 
| 975, 633      | 1000, 720     |
| 709, 463      | 1000, 0       |
| 575, 463      | 200, 0        |

I verified that my perspective transform was working as expected by drawing the polygon formed by the `src` and `dst` points onto the previously mentioned straight lines test image and its warped counterpart to verify that the lines appear parallel in the warped image. Here's the output:

![alt text][pt1] ![alt text][pt2]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code to identify the lane pixels and fit a polynomial to approximate the lane lines is in the IPython notebook located in "./solution.ipynb" in code cell titled "Lane Detection". The two main lane pixel identification functions are: slidingWindowLaneDetect() and targetedLaneDetect(). 

**Sliding Window Lane Detection**

In the slidingWindowLaneDetect function, I first compute a histogram of pixel intensities along the y-axis for each x-position, for the bottom half of the perspective transformed and thresholded image. This results in two distinct spikes corresponding to left and right lanes starting x-pixel positions as shown below for a test image:

![alt text][ld1]

I then start from this x-pixel position for each lane and define a small rectangular region of interest (window) to search for additional lane pixels. This window is eventually slid upwards from the bottom of the image and lane pixels are found. As I move the window up, I also re-center the window to the center of non-zero x-pixel positions (computed as mean) within the window. This enabled me to detect curving lanes in a given image. This process is illustrated below for the same test image from above:

![alt text][ld2]

I then grab all the pixels found by the sliding window method for the left and right lanes respectively (identified by figuring out which half of an image the pixels belong to) and pass it to the OpenCV polyFit() function to obtain the coefficients corresponding to a second order polynomial. I didn't see the need to go for any higher degree for the lane line fitting problem. The result is as follows for the above image:

![alt text][ld3]

**Targeted Lane Detection**

Once we have fitted lane lines for a given video frame image, we can perform a much targeted search for lane pixels around the previously identified lane lines as successive video frame images (for a sufficiently high frame rate) will likely be very similar. This is an optimization and also helps with reducing errors in lane detection between successive video frames. The code to do this is in the same cell in the "./solution.ipynb" IPython notebook under the targetedLaneDetect() function. Here the idea is to search around +/- 'M' pixels around the previously fitted lane lines for new lane pixels. The 'M' parameter is the search margin and I hand tuned it to an appropriate value. Once I have the lane pixels identified through this method, I then use OpenCV's polyfit method to get the new lane fits. Note that the function does some error correction which I will describe later in the Discussion section.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to compute the radii of curvature for the left and right lanes fit, and the vehicle position with respect to center, are in the "./solution.ipynb" IPython notebook under the code cell titled "Lane Detection". The getRadiusOfCurvatureInMeters() function computes the radius of curvature of fit using the formula outlined in http://www.intmath.com/applications-differentiation/8-radius-curvature.php. First, I compute the radius of curvature in the pixel space and then convert it into real world metric space by assuming a conversion factor for the amount in meters a given pixel maps to in the x and y direction.

The getVehicleDeviationFromLaneCenterInMeters() function works similarly by computing the vehicle deviation from lane center in pixels and then converting it into metric space using the same conversion factors. The logic to compute vehicle deviation is as follows: I assume that the camera is mounted in the center of the vehicle and that the center of the image it produces roughly corresponds to the position of the vehicle. I then compute the center of the lane using the left and right lane x pixel values at the bottom of the image. The difference between these two centers gives me the deviation of the vehicle from the lane center. A negative value indicates that the car is to the left of the lane center and vice versa.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code to plot the lane area, print the radii of curvature and vehicle deviation is located in the "./solution.ipynb" IPython notebook under the cell titled "Lane Detection" in function getImageWithLane(). I first draw the polygonal lane area formed by the left and right lanes (values generated using the polynomial fits passed as arguments to the functionn) on the warped image. I then use the inversePerspectiveTransformationMatrix to transform the warped image back to regular perspective from the car camera. I finally overlay this over the original undistorted image to produce the desired output image. Here is the output of this function for a test image (NOTE: The lane area doesn't extend all the way to the bottom of the image due to how I chose my original src and dst points while computing the perspective transformation matrix - this however does not impact the overall lane detection logic):

![alt text][lf1]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://youtu.be/TDjF1oiyKcc)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I spent a lot of time tuning the various parameters and trying out different combinations of the thresholding masks that would work for a wide range of scenarios including the challenge videos. However, I ran out of time to make lane detection work on the challenge videos. My pipeline has trouble with shadows, lane lines against a lighter background, different shades of asphalt on a single lane and direct sunlight shining into the camera. Once the term is over, I will return to the videos to see how I can tweak the parameters, use different color spaces and combine thresholding masks intelligently.

One interesting problem I had to solve with the targeted lane detection explained above was regarding high error rates between successive lane fits. In my implementation of targetedLaneDetect(), I also compute the error between the coefficients of the new fit and the previous fit of successive video frame images. For the main project video, the following plot shows the normalized absolute error in the polynomial coefficients for the left and right lanes for each successive video frame pair (for a polynomial: f(x) = A * y^2 + B * y + C, series 1 corresponds to error in A and so on and so forth):

![alt text][ld4]

As it can be seen, there are some very small number of extreme outliers. The 95th percentil values for the error is listed below (the values here are very small):

| 95%ile error  | Left Lane     | Right Lane   | 
|:-------------:|:-------------:|:-------------:
| Coeff. A      | 1.77          | 3.42         | 
| Coeff. B      | 1.26          | 1.80         |
| Const. C      | 0.08          | 0.02         |

This was causing my lane lines to jump around a lot. I solved this by using two techniques:

* First I use the normalized absolute error measure in the targetedLaneDetect function (in "./solution.ipynb" code cell titled "Lane Detection") to not return an outlier lane fit if it exceeds a tunable threshold; the previous lane fit is returned.
* Second, I use an exponentially weighted moving average smoothing for the lane fit coefficients (in "./solution.ipynb" code cell titled "Overall Pipeline" - function getSmoothedFits()). 

These two techniques resulted in producing a reasonably stable lane detection output as can be seen in the final output video.
