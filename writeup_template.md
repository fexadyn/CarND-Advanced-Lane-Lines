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

[image1]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/road_tfm.png "Road Transformed"
[image3]: ./output_images/thres_combined.jpg "Binary Example"
[image4]: ./output_images/warp.png "Warp Example"
[image5]: ./output_images/lane_pixels.jpg "Fit Visual"
[image6]: ./output_images/final_image.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./lane_detection.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

After obtaining camera matrix and distortion parameters from the previous step, I used OpenCV's undistort function to remove the distortion effect of the camera lens. Here is the effect of undistortion function applied on a checkboard image:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I have experimented with several different combinations of color and gradient thresholds. Because each thresholding step increases the processing time per frame, I wanted to use minimum number of preprocessing operations. My final thresholding approach is as follows:

1. Convert image to HLS color space and use S layer only.
2. Use gradient direction thresholding together with gradient magnitude thresholding
3. Use thresholding on S dimension only.
4. Combine (-OR) results from step 2 and 3

You can see detailed implementations of thresholding operations in the 6th block in the "./lane_detection.ipynb" file

Resulting thresholded image is as follows:
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `get_birdseye()`, which appears in the 8th code cell of the IPython notebook.  The `get_birdseye()` function takes as inputs an image (`img`), as well as camera calibration parameters. Hardcoded source and destination points to be used for perspective tfm can be seen in the same codeblock

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This is the main algorithm which detects lane pixels, fits second order polynomial and tracks lines for robust and smooth detection

For the first time detection of lane line pixels, I used the same approach suggested in the tutorial. First, calculated histogram of the lower half part of thresholded image on the horizontal axis. Identified peaks on the left and right half part of the histogram and used this two points as starting point for the search of lines. I used window search approach to find best window which contains the lines. Image is divided into 9 horizontal strips and windows are slided horizontally at each strip to maximize the number of white pixels within the frame of window. Finally, pixels reside in these windows are used as candidate lane pixels. In the next step, we fit 2nd order polynomial line to these pixels.

Detected line pixels on the thresholded image can be seen here:
![alt text][image5]

I created `Line()` class as suggested in the project page. In order to have a cleaner code I have implemented all line fitting, tracking and smoothing operations inside that class. Details of the implementation of `Line()` class can be seen in the 10th code block. So, only thing you need is to provide candidate line pixels to `Line()` object. As also suggested, if we have a successful detection from the provious iteration, we don't perform a blind histogram based search, instead we use the dilated version of the line to start our search within. 

In order to have smooth detection, I used averaging of latest 50 detections which corresponds to the last 2 seconds. Also, if the recent line detection is deviated than the average of the latest 50 detections a certain amount, I simply discard that detection. However, if we would have consecutive number of discarded detections, I simply delete all recent detections as well as start line pixel search from scratch. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this inside the implementation of `Line()` class which can be seen in the 10th codeblock.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `warp_back()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://youtu.be/x-SUS4qbc_0)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Lane detection works very well in the most part of the video. However, I mainly spent time on two challenging portions in the first video, portions where the road color is lighter. In those regions, lanes are harder to detect because of their similar color to the road surface. Improving thresholding methods and using tracking/smooting filter definitely helped in those regions. Implementing better filtering framework and improving threholding method can help further.
