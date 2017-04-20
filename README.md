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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

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

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

It's sufficient to perform this operation once after loading the notebook. I can then use the calibration coefficients for undistorting all images.

### Pipeline (single images)

I use two separate pipelines for the project_video and the challenge_video. It turned out that especially the need for image preprocessing steps was so different between these two scenarios that I had to split the pipelines up.

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

At the beginning of my jupyter notebook, I load a toolkit that offers all these operations. 

##### a. Color Transforms:

For example the functions H(img), L(img) and S(img) provide greyscale representations of the 3 channels in the HLS color space. I used 

TODO used where.

##### b. Gradients and thresholding

I have an implementation of all techniques described in the course in the jupyter notebook, such as abs_sobel_thresh or mag_thresh. I effectively ended up using abs_sobel_thresh in most cases. What's also useful, especially for the challenge_video, is simply thresholding the values of certain color channels. This is done with the thresh function after extracting a color channel (as described in the previous section)

TODO example of sobel, example of thresholded channel in challenge

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which takes as inputs an image (`img`). The source and destination points are set on top of the jupyter notebook in the global config. These parameters are called `SRC_TF` and `DST_TF`. I chose the hardcode the source and destination points in the following manner:

```python 
# Configuration
SRC_TF = np.float32([ [262.0, 680.0], [1042.0, 680.0], [701.0, 460.0], [580.0, 460.0] ])
DST_TF = np.float32([ [262.0, 720.0], [1042.0, 720.0], [1042.0, 0.0], [262.0, 0.0] ])
```

I verified that my perspective transform was working as expected by drawing the `SRC_TF` and `DST_TF` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

For the inverse transformation I use the function `inverse_perspective_transform`

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the code suggested in class. I had a sliding window approach `def sliding_window(binary_warped, histogram=None):` for initially identifying lane line pixels (based on a histogram of the lower half of the image) and also a second approach `def detect_from_previous(binary_warped, left_fit, right_fit):` that used existing polynomials to search lane line pixels in a specific area. 

The lane line pixels that were identified, were then used to fit 2nd order polynomials. Those were reported as lane lines and also fed to a filter that averaged over polynomials over several frames.

The following two images show the results of both types of calculations.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

happens in `def measure_curvature(fit, img):`
TODO

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `def draw_lane_undistorted(undistd, left_fit, right_fit):`. It takes an empty image and draws the polygon describing the lane in birds-eye view. Then this image is transformed back to the viewpoint of the vehicle camera and overlaid with the original undistorted image.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Overall:

* differnet code for different scenarios
* settings overfit to videos seen here
* no sanity checks
* no fallback to sliding window
* 

project_video:
- no big issues, gradx of L space then sliding window then detect from prev, filtering with hist size 50

challenge_video:
- left and rgith lane lines were extracted separately 
- left combination of thresholds of tttthe l and s spaces
- right: combination of threshold of l space and abs sobel of l space with threshold
- after that same functionality as in project video.

harder chalenge.
experiments with color spaces, but no real conclusion so far.
problems: too much light , too little light, tight curves where lane lines often cannot even be seen, sometimes i see reflections of the car on the windshield


* improvements: home-made video

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
