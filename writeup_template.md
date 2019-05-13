## Writeup
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
[image_calib]: ./examples/calibration.jpg "Calibration"
[image_undist]: ./examples/undist.jpg "Undistorted Image"
[image_thresh]: ./examples/thresh.jpg "Threshold"
[image_points]: ./examples/points_select.jpg "Points selection"
[image_warp]: ./examples/warped.jpg "Image Warped"
[image_fit1]: ./examples/fit1.jpg "Image fit 1"
[image_fit2]: ./examples/fit2.png "Image fit 2"
[image_fit3]: ./examples/fit3.png "Image fit 3"
[image_val]: ./examples/image_validation.jpg "Validation"

[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

This document is a substantial prove.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

This is the very first step to do. Cameras tend to have distortion because of the lens. To transform these curved shaped images to flat ones, at least 20 chessboard images taken from the camera are needed. Thes images must be taken from different angles and different distance.

In the camera_cal folder you'll find the images used to calibrate the camera.

The code for this step is contained in the first code cell of the IPython notebook located in "Project2".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image_calib]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image_undist]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image `In [5]`.
Firstly I decided to apply a Gaussian blur filter in order to make spots and shadows smooother. Then I converted the image to hls and kept the s channel. Separately I applied sobel and finally added this two thresholded images.
Here's an example of my output for this step from one of the crucial test cases. 

![alt text][image_thresh]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in `In [3]`.  The `warp()` function takes as inputs an image (`img`), as well as the region of interest (roi). The roi is meant to crop the image so that the warped image is cropped by:

```python
	img_warped[roi[2]:roi[3],roi[0]:roi[1]]
```
Where img_warped is the image after transformation.

To select the src points, I opened an image editor, select the 4 points as shown bellow and save the point's coordinates.
![alt text][image_points]

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 696, 455      | 700, 350       | 
| 1038, 675      | 700, 700      |
| 280, 675     | 500, 700      |
| 588, 455      | 500, 350       |

I verified that my perspective transform was working as expected by verifying that the lines appear parallel in the warped image.

![alt text][image_warp]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the sliding windows method. The first step is to calculate the histogram of the white pixels on the half bottom of the image, then find the peaks. The location of the peaks will be the starting point for the first window. I decided to set a margin of 40 pixels, as I eliminate most of noise in the thresholding step. The number of windows were set to 7.

In the images bellow you can see some examples of the fit to a polinomail.

![alt text][image_fit1]
![alt text][image_fit2]
![alt text][image_fit3]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
To reach this, firstly I converted pixels to meters. So when I calculate the polynamial in pixels, I also do it in meters `In [8]`.

```python
    left_fit_m = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
    right_fit_m = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
   ```
Then, to calculate the curvature for each line, I did the following.
```python
	 ((1 + (2*line_fit_m[0]*y_max + line_fit_m[1])**2)**1.5) / np.absolute(2*line_fit_m[0])
```
Finally, to get the distance to the center I denied a function called get_distance(),where I did the following calculation:
```python
    # Calculate vehicle center
    xMax = img_shape[1]*xm_per_pix
    yMax = img_shape[0]*ym_per_pix
    vehicleCenter = xMax / 2
    lineLeft = left_fit_m[0]*yMax**2 + left_fit_m[1]*yMax + left_fit_m[2]
    lineRight = right_fit_m[0]*yMax**2 + right_fit_m[1]*yMax + right_fit_m[2]
    lineMiddle = lineLeft + (lineRight - lineLeft)/2
    diffFromVehicle = lineMiddle - vehicleCenter
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step by `In [9]`. Here is an example of my result on a test image:

![alt text][image_val]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/out0.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

1. When the floor changes of color, it does not perform the best. The cause is that I did not implement search around the previous calculated polynomial. So I don't track the lane detection, and this also causes a computational time cost. 

2.For lighting conditions, I would like to test methods like CLAHE and Gamma correction, in order to normalize the lighting conditions.

3.My principal goal was to reach a good performance on the challenge videos, however I did not get the time and I should move forward on the other projects. One solution for the challenge videos would be the use of deep learning. As the position of the car is almost centered the most of the time, it would be easy to get labeled data to train the model. Once I completed the first term, I'm sure I'm gonna get back an try this. 
