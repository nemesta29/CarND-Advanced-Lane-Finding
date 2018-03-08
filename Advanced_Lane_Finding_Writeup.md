
## Project Writeup

### The writeup below shall explain the thought process and steps followed in making this project

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

[image1]: ./output_images/calib_calibration2.jpg "ChessBoard"
[image2]: ./calibration1.jpg "Distorted"
[image3]: ./output_images/undist_calibration1.jpg "Undistorted"
[image4]: ./output_images/warped_test6.jpg "Binary Example"
[image5]: ./output_images/perspect_test6.jpg "Perspective"
[image6]: ./output_images/lane_test6.jpg "Lane Lines"
[image7]: ./output_images/final_test6.jpg "Final Image"
[video1]: ./Project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

## Writeup 

### Camera Calibration

#### 1. Calibration of camera and undisortion of images

The code for this step is contained in the second code cell of the Jupyter notebook located in "Advanced_Lane_Finding.ipynb" under `calibration_buff()` and `calibrate()` functions   

**calibration_buff()** : I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `obj_pts` is just a replicated array of coordinates, and `object points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `image points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. The function `cv2.findChessboardCorners()` was used to detect *9x6* corners on various chessboard images

![alt text][image1]

**calibrate()** : Here I use the `object points` and `image points` generate to create a translation matrix and distortion coefficients to correct distortion due to camera error. `cv2.calibrateCamera()` function was used to obtain these parameters.

### Pipeline (single images)

#### 1. Correcting distortion due to camera error
**undistort()** : The matrix and coefficients were then used to undistort any given image taken from the camera. This function used `cv2.undistort()` to correct the distortion in images as shown below:

Distorted Image : 
![alt text][image2]
Undistorted Image :
![alt text][image3]

#### 2. Gradients, color spaces and transformations used to transform the image to a binary image and identify the lane lines better.

I used the function `color_grad()` provided in the second cell of my pipeline to take gradient of the image. I used a *Sobelxy* transform to take one image gradient. Alongwith that, through experimentation, I came upon the HLS color scheme to further filter data.
I used the *L* channel to form a threshold against dark edges, so that no shadow would be considered a lane. I also used the *S* channel to pick lanes irrespective of lighting conditions and brightness.
I combined the two channel gradients using bitwise AND, and combined the resultant with Sobel gradient using bitwise OR. The final binary output obtained was something like this:

![alt text][image4]

#### 3. Perspective transform - used to change the image to a bird's eye view of the same

The function `perspective()` located in the second cell of the Jupyter notebook, is used to transform the images to a bird's eye view of the road. This function takes as input the image and it's transformation matrix obtained using `cv2.getPerspectiveTransform()`. Since the same source and destination points shall be used, the call for transformation matrix was made only once in code cell 4.
The source and destination points were selected by trial and error, till a suitable transform was obtained, that could be used to properly identify the lanes and the values so obtained were

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 560, 460      | 120, 0        | 
| 100, 700      | 80, 720       |
| 1120, 700     | 1100, 700     |
| 740, 460      | 980, 0        |

*I used a region slightly larger than the lane lines to allow for curvature of road. This helped improve the final result obtained during video analysis

Running the matrix so obtained through the given image, the function performed transformation using `cv2.warpPespective()` to provide the following result for the image tranformed in the previous step:

![alt text][image5]

#### 4. Description of how I identified lane-line pixels and fit their positions with a polynomial

I then began my procedure for finding the lane lines in the image, with the code provided in the classroom as my starting point. The function `lane_find`  is used to detect lane lines in the image. It takes as input a binary transformed image and identifies the lanes.
The code begins by drawing a histogram of the image. Since by now we have a binary image, with reduced noise, the lanes should be the highest peaks in the histogram. Using this we identify the mid-point of the image. Points upto the the mid-point are classifed as *left* and those greater than the mid-point are considered *right*. 
A sliding window search is then employed and the non-zero values obtained within this region was obtained. The point sto the left were saved as good left indices and the points to the right of midpoint were saved as good right indices. These points were then plotted in the arrays `left_fitx` and `right_fitx`.
The function `visualise()` was used to then map the same points on the image and identify the lane lines as shown below

![alt text][image6]

#### 5. Calculation of radius of curvature and position of car from the center of the image

The radius of curvature and position of the vehicle from centre were calculated in the function `calc_curve()` present in the second cell of the notebook. I used the following scaling factors while calculating the radius and postion of the vehicle in the pipeline
    ym_per_pix = 30/720 
    xm_per_pix = 3.7/700 

For the radius of curvature, a polynomial equation was fitted through the location points of the lanes obtain to form a polynomial in y. The radius was obtained using a formula R = ((1 + (dx/dy)^2)^(3/2))/(d2x/dy2).
Code provided in the classroom material was used as a starting point in this process, calculating the radius against the lanes so plotted. The curve for both left and right lanes was obtained, which was averaged out for the final radius of curvature.
For calculating the position of the image, the mid-point of the lanes was first obtained by taking mean of the base plots of both the lanes. The x-coordinate of this pixel value was then subtracted from 640 which was the center of the image. If the value was negative, the car was assumed to be at *left of center* and if positive, it was assumed to be at *right of center*

#### 6. An example image of my result plotted back down onto the road such that the lane area is identified clearly.

Using the code provided in the tips and tricks section of the classroom, the function `draw_region` is used to retransform the image back to normal view and plot the lane region on the road surface. It takes the image, location of points and an inverse perspective transformation matrix obtained by switching the `src` and `dst` points. The final image obtained with the lane lines marked is as follows

![alt text][image7]

---

### Pipeline (video)

#### 1. The final video with the lanes identified and marked, with a slight margin of error, can be found at the link below
Here's a [link to my video result](./Project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
* The first problem area I find with my code are the hardcoded pixel values for transformation matrix. These values shall work for the present scenario, but the moment a sudden bump or curve arrives, they may lead to a drastic change in the lane region as the ratio of camera angle to road changes briefly. The solution to this would be to dynamically identify the `src` and `dst` points

* The code identifies lane lines based on thresholding of gradients and color spaces. While this is effective, it can also detect distinct borders as edges and plot them as lines. This can lead to incorrect identification of lane lines. To rectify this we may include more sanity checks and averaging of lane lines, so that abrupt deviation from the current position may be avoided and lanes be correctly identified

