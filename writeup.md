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

[image1]: ./output_images/chessboard_calibration.png "Chessboard Calibration"
[image2]: ./output_images/02-undistort_chessboard.png "Undistorted Chessboard"
[image3]: ./output_images/road-transformed.png "Road Transformed"
[image4]: ./output_images/binary_combo_example.png "Binary Example"
[image5]: ./output_images/sliding_window_polyfit.png "Sliding Window Polyfit"
[image6]: ./output_images/sliding_window_histogram.png "Sliding Window Histogram"
[image7]: ./output_images/polyfit_from_previous_fit.png "Polyfit Using Previous Fit"
[image6]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image7]: ./output_images/example_output.jpg "Output"
[image8]: ./output_images/draw_lane.png "Data shown onto Original Image"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first two code cell of the IPython notebook located in "./advanced_lane_detection.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The image below depicts the results of applying `undistort` to one of the project images:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. But then I selected the L channel of the HLS color space to detect white lines and the B channel of the LAB colorspace to seperate yellow lines. Here's an example of my output for this step.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is written in the 9th cell, which takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 450, 0        | 
| 707, 720      | w-450, 720    |
| 258, 682      | 450, h        |
| 1049, 682     | w-450, h      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial using two functions "sliding_window_polyfit" and "polyfit_using_prev_fit".

![alt text][image5]

![alt text][image6]

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the function named curvature_offset().
To calculate radius of curvature, I used this formula:

```
curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])
```

To calculate position of vehicle, I used this formula:

```
lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
```


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function named 'draw_lane', also I have shown the data such as raduis and position of car on the image, using function "write_text_on_image".  Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Actually, it was little bit hard to find which color space to use and which varient of sobel gradient as there were different lighting conditions. But after that, it was not much difficult to identify the threshold values of the colorspaces. Also the B channel from LAB colorspace seperates the yellow color very well. The pipeline worked well in this type of video input, but when the lane lines don't necessarily occupy the same pixel range values on this video then it would create problems when the white lines didn't contrast with the rest of the image enough(eg. white car). 
