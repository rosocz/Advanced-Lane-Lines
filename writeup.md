## Writeup Template

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

[image1]: ./output_images/undist_calibration.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/color_transform.png "Binary Example"
[image_polynom]: ./output_images/raw_polynom.png "Ploted curve"
[image_warped]: ./output_images/straight_warped.png "Testing warped"
[image_grad]: ./output_images/gradients.png "Gradient variants"
[image_lane_lines]: ./output_images/single_image_result.png "Draw lane lines on original"
[video1]: ./video_output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used color transformation, I have a function called `colorTransform` where I apply white filter to get all pixel whith white color above the threshold. Than I used yellow filter and did the same. For yellow filter I tried gamma correction to get lighter input to solve images where yellow line is in shade.
Result of white and yellow filter is reduced to grayscale and another threshol is applied to select white pixels only.
The output is quite good, I used saturation channel (HSL) and value channel (HSV) just to help with some complicated areas but  I don't feel it was absolutely necessary.

![alt text][image3]

I created functions to produce gradient images (x, y, magnitude and direction) I used combination of x, y and magnitude.

![alt text][image_grad]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

To be able transform image I had to find correct parameters of transformation to get right transformation matrix. This is done by using image where lane lines are more or less straight. I was just trying to find best values to get image with parallel lane lines.

Here you can see result of testing transformation

![alt text][image_warped]

So when I need to warp any image I created function `warp` which uses still same transform matrix and it is done. Camera position is same for all images so transform matrix is same as well.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I created function `getLines` where there is series of actions to get final polynomial.
* get histogram of image to identify position of lane line. We have binary image so the task is quite easy
* find center of image, get left and right part, find highest value of pixel for each side
* that's the starting point to draw rectangle
* rectangle works as area where we are looking for all white pixels
* if the input image is correct, white pixels should form something like shape of lane line and we are able to draw curve among pixels
* found relevant pixels are saved to the list of pixels which are used to draw final curve
* then it's required to identify next starting poin and do the same for next rectangle moved by height of the first one

It's repeated until the end of the image. The result is list of pixels on lane lines.

As I said all tasks are done for left and right part separately so the result of curve ploted among found pixels looks like this.

![alt text][image_polynom]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Curvature of lane line is counted by function `getCurvature`. It counts curvature in meters, by counting in parameters `ym_per_pix` and `xm_per_pix` we are able meassure curvature from pixel dimensions to real world meters.
Counting distance of vehicle from the center is part of the function `getLabeledImage`. We are able to get points on curves so we can caunt center between them and we can expect that middle of the image acts as position of the vehicle.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is done by calling functions `getLines`, `addLineToImage`, `getLabeledImage`.
* I find lines on the single image 
* combine with original picture
* add info about curvature and vehicle position

![alt text][image_lane_lines]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

For smooth progress of the lane line drawing I used very simple buffer of last 17 images, about 3/4 second to avoid rapid changes.
To prevent any unexpected curves I'm skipping all curves with curvature which is very different to average of current buffer.

Here's a [link to my video result](./video_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Preprocessing part of images was challenging until I found hint with white and yellow filters. Using only rgb, hsv, and hls channel was not possible for me.

Another issue I had when I was analysing video without buffer, that was really poor result. Than it was just a question of balancing the best parameters of buffer.

I tried challenging videos as well, that was not good, it failed because it lost all points for one frame, I could use buffer and tolerate cases where no point are found on image. Another option is to improve preprocessing so we can avoid cases where we have no data.