# CarND-Advanced-Lane-Lines-mwolfram



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

[sobel]: ./output_images/straight_lines1_sobel.jpg 
[sw]: ./output_images/test2_sliding_window.jpg
[persp]: ./output_images/straight_lines1_persp.jpg
[thr]: ./output_images/straight_lines1_thresholdedL.jpg
[undist_calib]: ./output_images/undistorted_calibration1.jpg
[persp_sobel]: ./output_images/straight_lines1_persp_sobel.jpg
[dprev]: ./output_images/test2_detect_from_previous.jpg
[undist_straight]: ./output_images/undistorted_straight_lines1.jpg
[persp_thr]: ./output_images/straight_lines1_persp_thresholdedL.jpg
[orig_with_lane]: ./output_images/test2_lane_orig.jpg
[video1]: ./project_video_final_out.mp4 "Project Video"
[video2]: ./challenge_video_final_out.mp4 "Challenge Video"
[video3]: ./harder_challenge_video_final_out.mp4 "Harder Challenge Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
### Camera Calibration

All Code is located in a Jupyter notebook located at ./pipeline.ipynb

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I used the function provided in class. The implementation can be found in the Jupyter notebook in the function 

```python
def calibrate(nx, ny):
```

The function is called in a separate cell:

```python
# Calibrate
ret, mtx, dist, rvecs, tvecs = calibrate(9, 6)
```

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to a test image using the `cv2.undistort()` function and obtained this result: 

![Undistorted test image][undist_calib]
*Undistorted test image*


It's sufficient to perform this operation once after loading the notebook. I can then use the calibration coefficients for undistorting all images.

### Pipeline (single images)

I use two separate pipelines for the project_video and the challenge_video. It turned out that especially the need for image preprocessing steps was so different between these two scenarios that I had to split the pipelines up.

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![Undistorted image of straight lane lines][undist_straight]
*Undistorted image of straight lane lines*

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

At the beginning of my jupyter notebook, I load a toolkit that offers all these operations. 

##### a. Color Transforms:

For example the functions `H(img)`, `L(img)` and `S(img)` provide greyscale representations of the 3 channels in the HLS color space. I used the `L(img)` function before passing an image to the sobel filter. Also I postprocessed L color channels by simply thresholding them. This helped me extract left and right lane lines in the challenge_video.

##### b. Gradients and thresholding

I have an implementation of all techniques described in the course in the jupyter notebook, such as abs_sobel_thresh or mag_thresh. I effectively ended up using abs_sobel_thresh in most cases. What's also useful, especially for the challenge_video, is simply thresholding the values of certain color channels. This is done with the thresh function after extracting a color channel (as described in the previous section)

![Sobel in X direction, thresholded][sobel]
*Sobel in X direction, thresholded*

![L color channel, thresholded][thr]
*L color channel, thresholded*

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which takes as inputs an image (`img`). The source and destination points are set on top of the jupyter notebook in the global config. These parameters are called `SRC_TF` and `DST_TF`. I chose the hardcode the source and destination points in the following manner:

```python 
# Configuration
SRC_TF = np.float32([ [262.0, 680.0], [1042.0, 680.0], [701.0, 460.0], [580.0, 460.0] ])
DST_TF = np.float32([ [262.0, 720.0], [1042.0, 720.0], [1042.0, 0.0], [262.0, 0.0] ])
```

I verified that my perspective transform was working as expected by drawing the `SRC_TF` and `DST_TF` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Straight lines image warped to birds-eye view][persp]
*Straight lines image warped to birds-eye view*

![Straight lines image warped to birds-eye view, after applying sobel][persp_sobel]
*Straight lines image warped to birds-eye view, after applying sobel*

![Straight lines image warped to birds-eye view, after applying threshold on L channel][persp_thr]
*Straight lines image warped to birds-eye view, after applying threshold on L channel*

For the inverse transformation I use the function `inverse_perspective_transform`

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the code suggested in class. I had a sliding window approach `def sliding_window(binary_warped, histogram=None):` for initially identifying lane line pixels (based on a histogram of the lower half of the image) and also a second approach `def detect_from_previous(binary_warped, left_fit, right_fit):` that used existing polynomials to search lane line pixels in a specific area. 

The lane line pixels that were identified, were then used to fit 2nd order polynomials. Those were reported as lane lines and also fed to a filter that averaged over polynomials over several frames.

The following two images show the results of both types of calculations.

![Sliding Window approach on test2.jpg][sw]
*Sliding Window approach on test2.jpg*

![Detecting from previous fits on test2.jpg][dprev]
*Detecting from previous fits on test2.jpg*

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

These two calculations happen in `def measure_curvature(fit, img):`and in `lane_offset(left_fit, right_fit, img):`. The curvature calculation is taken from the course. I apply the following function to convert from polynomial coefficients to radius: 

```python
px_curverad = ((1 + (2*fit[0]*y_eval + fit[1])**2)**1.5) / np.absolute(2*fit[0])
```

I do the same thing for the radius in meters, with a factor applied.

The lane_offset is calculated by finding the bottom-most points of a fit in x direction for both lane lines, averaging between them to find the lane center and subtracting the actual center of the image. After applying the same conversion factor as before I get the offset in meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `def draw_lane_undistorted(undistd, left_fit, right_fit):`. It takes an empty image and draws the polygon describing the lane in birds-eye view. Then this image is transformed back to the viewpoint of the vehicle camera and overlaid with the original undistorted image.

![Resulting image with marked lane][orig_with_lane]
*Resulting image with marked lane*

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_final_out.mp4), as well as to the result of the [challenge_video](./challenge_video_final_out.mp4).

There is also a link to the [harder_challenge_video](./harder_challenge_video_final_out.mp4), however those results are not good enough and just meant to demonstrate the difficulties that the pipeline faces (also discussed below). For this video I used the same pipeline as for the project_video, with sliding window in every iteration and a history size of 10.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

##### Shortcomings of the current pipeline(s):

* Different pipelines used for different scenarios. There is no single pipeline that can handle both or even all three scenarios.
* Settings are overfit to videos. In different lighting conditions or with different lane colors, we'd probably run into problems.
* Filtering is used, however there's never a sanity check, mainly because the existing pipeline already worked nicely with the videos provided. Without a sanity check, I never decide to skip measurements or to fall back to sliding window.

##### a. Project Video:

The project video did not present too big difficulties. I used the gradient in x direction of the L color channel, then used an initial sliding window run and the detection from previous fits within a certain margin took it from there. What was extremely useful was taking the average fit of the last 50 frames.

##### b. Challenge Video:

Was, as the name suggests, a little trickier. I ended up handling the left and right lane lines differently, extracting them with precisely chosen thresholds of the L and S channels and their gradients. To be exact:

* Left line is a combination of the L and S spaces, thresholded
* Right line is a combination of the thresholded L space and also the absolute sobel of the L space, again thresholded.

When it comes to detecting lane line pixels I used the same approach as for the project video.

##### c. Harder Challenge Video:

This one was too tricky for now. Problems:

* I experimented with color spaces, but had no real conclusion as to which ones would yield the best results
* Too much light, too little light, and this is changing. Also probably a reason why toying with color spaces and thresholds did not lead anywhere.
* Tight curves that often can't be seen.
* Reflections of the car on the windshield

##### Possible improvements: 

* Take own video, test with that
* Introduce sanity check (for example difference in curvatures of both lane lines)
