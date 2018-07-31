
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

[original]: ./output_images/camera_calib/original.jpg
[undistorted]: ./output_images/camera_calib/undistorted.jpg "Undistorted"
[test_original]: ./test_images/test2.jpg
[test_undistorted]: ./output_images/undist/test2.jpg "Road Transformed"
[image3]: ./output_images/sobel_s_channel/test2.jpg "Binary Example"
[before_warp]: ./output_images/sobel_s_channel/straight_lines2.jpg
[warped]: ./output_images/threshold_warped/straight_lines2.jpg
[lane_pixel]: ./output_images/lane_pixel/test2.jpg
[marked_lane]: ./output_images/marked_lane/test2.jpg
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This is what you are reading now!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the IPython notebook located in "./advancedLaneFinding.ipynb".  

I first defined 'objp', which contains the cordinates of each corner of a chessboard on a flat sarface. Then, I collect the imgpoint by using cv2.findChessboardCorners() on gray scaled image. Everytime I find chessboard corners correctly, I keep track of the same number of objp to caliculate camera matrix and distortion coefficients with cv2.calibrateCamera(). The following the result of my camera calibration.

![alt text][original]
![alt text][undistorted]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Based on the camera calibration I got on the previous step, I undistort each test image.
Here's one example.

original
![][test_original]
undistorted
![alt text][test_undistorted]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used saturation channel and lightness channel in HLS color space to detect lane line pixels in addition to sobel x gradient.
(thresholding steps at lines 4 through 34 in in third code cell in `advancedLaneFinding.ipynb`).  Here's an example of my output for this step. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. 

First, I look for 4 points that craates neat trapezoid on the undistorted version of a straight lane image by hand, and then choose the coordinates to transfer the original trapezoid into a rectangle. With the matrix transform obtained by cv2.getPerspectiveTransform(), I warp the image into the bird's eye view image. (5th code cell, line 22-32, ./advancedLaneFinding.ipynb)

Here's how the coordinates look like in code

```python
src = np.float32([[596, 450], [690, 450], [1106, 720], [218, 720]])
dst = np.float32([[width//4, 0], 
                  [width//4*3, 0],
                  [width//4*3, height],
                  [width//4, height]])
```

So the destination rectangle has the same height and the half width of the original image.

To make sure my prospective transform is working, I tested on the straight lane image and checked if the transformed lanes are parallel.

before transform
![alt text][before_warp]
warped
![alt text][warped]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To find lane pixel, I used sliding window technique. As a starting point, I calculate the histogram of the bottom half of the image to identify each peak in left half and right half of the image as the points where lanes start. Then I search for lane pixels in the rectanglar area around the starting points. If sufficient amount of pixels are found, I take its mean and adjust next starting point. I keep repeating this step until I reach the top of the image and encapsulate this into a function called find_lane_pixels() (3rd code cell, line 36-95, ./advancedLaneFinding.ipynb). With the pixel found through this process, I can call fit_polynomial() (3rd code cell, line 123-132, ./advancedLaneFinding.ipynb).
Here's one of the results

![alt text][lane_pixel]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculate the radius of curvature with the formula given in the lesson in terms of meter (3rd code cell, line 149-156, ./advancedLaneFinding.ipynb).
I calculate the position of vehicle by calculating the difference between the center pixel in the image and the center of the two lanes (3rd code cell, line 160-163, ./advancedLaneFinding.ipynb).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this pipline on 5th code cell in ./advancedLaneFinding.ipynb.

\* positive value of position means the vehicle is shifted to the right and negative value means it's shifted to the left.

![alt text][marked_lane]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a link to my video result
[On github](./output_videos/marked_project_video.mp4)
[On youtube](https://youtu.be/7qnh5_HvZZ0)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
