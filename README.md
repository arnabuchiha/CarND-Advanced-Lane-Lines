# Advanced Lane Finding Project
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

This project utilizes a software pipeline to identify the lane boundaries in a video.

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


The images for camera calibration are in the `camera_cal` folder.  The images in `test_images` are used in testing the pipeline on single frames.  Examples of the output from each stage of the pipeline are the `ouput_images` folder.  The video `project_video_output.mp4` is target video for the lane-finding pipeline.  Each rubric step will be documented with output images and usage.

# [Rubric Points](https://review.udacity.com/#!/rubrics/571/view)

## Camera Calibration
1) Have the camera matrix and distortion coefficients been computed correctly and checked on one of the calibration images as a test?

The code for this step is contained in `camera_calibration.ipynb`

This notebook loads calibration images of chessboards taken at different angles.  Each image is grayscaled and sent into `cv2.drawChessboardCorners`.  The resulting "object points" are the (x, y, z) coordinates of the chessboard corners in the world.

For each set of corners, the output is displayed and shown to the user (if specified).  The image is also saved to disk (if specified).
  
Finally the corner points are sent to `cv2.calibrateCamera` to get resulting image points and object points.  This dictionary is then saved for reuse in undistorting other images in the pipeline.

![](output_images/chessboard1.jpg) ![](output_images/chessboard9.jpg)

##### Example chessboard images with corners drawn

## Pipeline(Image)
This sections includes all scripts and functionality to process images throughout the lane-finding pipeline.
## You can find the code for the following in `final_pipeline.ipynb` with their respective heading

### 1) Provide an example of a distortion-corrected image.
This is provided in the `camera_cal_output` directory.

![image1](https://raw.githubusercontent.com/arnabuchiha/CarND-Advanced-Lane-Lines/master/camera_cal_output/undistorted_cal1.jpg)


### 2) Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.
I explored several combinations of sobel gradient thresholds and color channel thresholds in multiple color spaces. These are labeled clearly in the Jupyter notebook. Below is an example of the combination of sobel magnitude and direction thresholds.
You can view them in the notebook `final_pipeline.ipynb`
![binary_image](https://raw.githubusercontent.com/arnabuchiha/CarND-Advanced-Lane-Lines/master/output_images/binary_image.jpg)

### 3) Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In order to fit the lane lines, the image is transformed into the "birds-eye" perspective. The function that performs this transformation is

perspective_transform(image_binary,src,dst)
where src defines the source points in the original image and dst defines the destination points in the "bird-eye" image.

### 4) Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial.
The functions sliding_window_polyfit and polyfit_using_prev_fit, which identify lane lines and fit a second order polynomial to both right and left lane lines, are clearly labeled in the Jupyter notebook as "Sliding Window Polyfit" and "Polyfit Using Fit from Previous Frame". The first of these computes a histogram of the bottom half of the image and finds the bottom-most x position (or "base") of the left and right lane lines. Originally these locations were identified from the local maxima of the left and right halves of the histogram, but in my final implementation I changed these to quarters of the histogram just left and right of the midpoint. This helped to reject lines from adjacent lanes. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy polyfit() method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works.
![image](https://raw.githubusercontent.com/arnabuchiha/CarND-Advanced-Lane-Lines/master/screenshots/marked_lane.png)

![histogram](https://raw.githubusercontent.com/arnabuchiha/CarND-Advanced-Lane-Lines/master/screenshots/histogram.png)



### 5) Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is determined by the gvien formulas in this https://www.intmath.com/applications-differentiation/8-radius-curvature.php


The position with respect to center can be calculated by evaluating the fitted function at the bottom of the image and compares that to the center of the image. The curvature and the position can be calculated using the function

`calc_curv_rad_and_center_dist(bin_img, l_fit, r_fit, l_lane_inds, r_lane_inds)`

### 6)Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![image](https://raw.githubusercontent.com/arnabuchiha/CarND-Advanced-Lane-Lines/master/screenshots/marked_road.png)

## Pipeline(video)
### 1)Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!)

This video is in main directory named `project_video_output.mp4`

https://youtu.be/xLF-YRNAYC8
## Discussion

My thinking on this project is similar to project 1.  It's a fascinating challenge that taught me quite a bit more about CV techniques.

But I think the solution in general is heuristic and is not optimal for general lane-finding.  As seen in the challenge videos, changes in road conditions and colors present a problem and must have specific color filters.  Also, other lines are picked up by the pipeline that match all the processing but are not the lane lines.

Similarly, during lane-change, lane-merging or any other situation where lanes are converging to normal pattern, the algorithm will fail.

Finally, many roads don't have lane markings, or the markings are very faint or broken up so determining a path would fail as well.

Ultimately, lane-finding might need to have a more nuanced solution, with heuristic camera techniques used for well-known roads.  Combined with geospatial, temporal and other variables might alter threshold values for filters automatically.  Conversely, if lane-finding can be a problem solved by deep learning, then CV techniques could be moot (or used in some fused manner).