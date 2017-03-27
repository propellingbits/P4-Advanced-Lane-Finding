##Writeup Template

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./examples/Thresh-bin-2.png "Road Transformed"
[image3]: ./examples/combined-threshold.png "Binary Example"
[image4]: ./examples/persp-trans.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"
[image7]: ./examples/lane-detection.png "Pipeline Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./exP4-Advanced-Lane-Finding/p4-final.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell # 14 in my notebook).  Here's an example of my output for this step.  

I ran a test (cell #187) on test-images provided by udacity to check the quality of my pipeline (cell # 11). For results, please refer to second image below.

![alt text][image7]
![alt text][image4]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform can be found at cell # 187 in my notebook. I am passing the test images to pipeline. I chose to hardcode the source and destination points in the following manner:

```
src = np.float32([(575,464),
                    (707,464), 
                    (258,682), 
                    (1049,682)])
dst = np.float32([(450,0),
                    (w-450,0),
                    (450,h),
                    (w-450,h)])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 450, 0        | 
| 707, 464      | 750, 0        |
| 258, 682      | 450, 720      |
| 1049, 682     | 750, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

My lane lines with a 2nd order polynomial looks like following image. Code for the same can be found in notebook at cell #3 (sliding_windows())

I will say this was the toughest part of this project for me. I played with HSV and HLS color spaces. Also played with gradients and sobel operators. It did not give me satisfying result. Along the way, I learnt, to begin with, the input image must be undistorted one and it must also be warped before proceeding further. After this, I thresholded channel L of HLS and channel B of LAB color space. At the end, I combined the result of both of these operations. I arrived here after some failures, experiments and searches. Corresponding code can be found in cell # 181 (finalPipeline()).

For fitting a polynomial to the detections, I took reference from lessons and code. In cell number 24, output from pipeline is passed on to sliding_window_polyfit function. This function first draws a histogram on lower half of the images. It does a good job of drawing the peaks where we have lane lines as it is a binary image and we have 1 values where lane lines are located.  This was used as a starting point for where to search for the lines. From that point, I slided a window, placed around the line centers, to find and follow the lines up to the top of the frame.

![alt text][image7]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell # 247 (measuringCurvature()) in my notebook.

In the sliding_windows() function, I have located the lane line pixels, used their x and y pixel positions to fit a second order polynomial curve:

f(y)=Ay​2+By+C

We are fitting for f(y), rather than f(x), because the lane lines in the warped image are near vertical and may have the same x value for more than one y value.

The y values of image increase from top to bottom, so if, for example, we wanted to measure the radius of curvature closest to our vehicle, we could evaluate the formula above at the y value corresponding to the bottom of your image, or in Python, at yvalue = image.shape[0].

The radius of curvature is based upon [this link](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) and calculated in cell # 247 (measuringCurvature()) using below code line :
```
curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])
```
In this example, `fit[0]` is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and `fit[1]` is the second (y) coefficient. `y_0` is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). `y_meters_per_pixel` is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters. 

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:
```
lane_center_position = (r_fit_x_int + l_fit_x_int) / 2
center_dist = (pov_wrtcl - lane_center_position) * xm_per_pix
```
`r_fit_x_int` and `l_fit_x_int` are the x-intercepts of the right and left fits, respectively. This requires evaluating the fit at the maximum y value (719, in this case - the bottom of the image) because the minimum y value is actually at the top (otherwise, the constant coefficient of each fit would have sufficed). The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is available to view at cell # 256 in my notebook. It calls prcess_image function which internally call the pipeline:

![alt text][image2]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had lot of trouble in mapping the detected lane line over to original image. I have not tested my pipeline on the challenging video. In case, it fails then I would like to improve it. Some of the students have implemented a debugging pipeline and some filters tool which I would also like to add to my codebase.

