## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/Camera_cal.png "Camera Calibration"
[image2]: ./examples/Undist_Checkerboard.png "Undistorted Checkerboard"
[image3]: ./examples/Undist_image.png "Undistorted Image"
[image4]: ./examples/BirdsEyeView.png "Warp to BEV"
[image5]: ./examples/Color_Channels_BEV.png "Color Channels on BEV"
[image6]: ./examples/Sobel_Abs.png "Sobel_Abs"
[image7]: ./examples/Sobel_Mag.png "Sobel_Mag"
[image8]: ./examples/Threshold_Grad_Dir.png "Threshold_Grad_Dir"
[image9]: ./examples/Threshold_S.png "Threshold_S"
[image10]: ./examples/HLS_L_Channel.png "HLS_L_Channel"
[image11]: ./examples/LAB_B_Channel.png "LAB_B_Channel"
[image12]: ./examples/Pipeline_tests.png "Pipeline_tests"
[image13]: ./examples/Pipeline_images.png "Pipeline_images"
[image14]: ./examples/LaneLine_Boxes.png "LaneLine_Boxes"
[image15]: ./examples/Histogram.png "Histogram"
[image16]: ./examples/LineFit.png "LineFit"
[image17]: ./examples/LaneDetectionOnRaod.png "LaneDetectionOnRaod"
[image18]: ./examples/LaneDetectionOnRaod_wText.png "LaneDetectionOnRaod_wText"
[video1]: ./output_images/project_video_output.mp4 "Project Video"
[video2]: ./output_images/challenge_video_output.mp4 "Challenge Video"
[video3]: ./output_images/harder_challenge_video_output.mp4 "Harder Challenge Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./AdvancedLaneLineDetection.ipynb.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the checkerboard and test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]
![alt text][image3]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result. 

I used a combination of color and gradient thresholds to generate a binary image as seen in code cells 7-26 of the IPython notebook  Here's a few example of my output for this step.

![alt text][image5]
![alt text][image6]
![alt text][image7]
![alt text][image8]
![alt text][image9]
![alt text][image10]
![alt text][image11]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `corners_unwarp()`, which appears in code cell 5 in the Ipython notebook. The `corners_unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the outer for detected corners for the source points and for the destination points, I arbitrarily chose some points to be a nice fit for displaying the warped result:

```python
src = np.float32([(575,464),
                  (707,464), 
                  (258,682), 
                  (1049,682)])
dst = np.float32([(450,0),
                  (w-450,0),
                  (450,h),
                  (w-450,h)])
```


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and verified that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the functions sliding_window_polyfit and polyfit_using_prev_fit to identify lane lines and fit a second order poly to both right and left lane lines. This can be found in code cells 27 and 31 of my IPython notebook.

This starts by computing the histogram of the bottom half of the image and to find the starting points for both the L and R lane line at the bottom of the image to the left and right of the horizontal center of the image. 

![alt text][image15]

I then divided the image into 10 windows to id lane pixels, each consecutive window is centered on the midpoint of the previous one, allowing the windows to follow the lane lines. All pixels identified to each lane line are inputs to the Numpy polyfit() method and a second order poly is fitted to the pixels. The image below shows sthe windows and polyfit.

![alt text][image14]

The polyfit_using_prev_fit uses the previous fit as a starting point and searches for lane pixels within a certain range of the previous polyfit.

![alt text][image16]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated in code cell 33 with calc_curv_rad_and_center_dist. It uses the second order polynomial fit and the meters_per_pixel to calculate the curvature. The meters_per_pixel for the y dir was calculated by counting the number of pixels for a standard lane line length of 10 ft and in the x dir for a standard lane width of 12 ft.

The position of the vehicle with respect to the center of the lane was calculated based on the distance between the center of the lane and the image center. The center of the lane was calculated by evaluating the L and R polynomials at the bottom of the image. 


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 35 with draw_lane in the jupyter notebook. A polygon is generated from the left and right poly fits, then warped back to the original image using Minv, the inverse perspective matrix.

![alt text][image17]

Test showing the radius of curvature and center offset was added to the original image.

![alt text][image18]
---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

Here's the challenge video link, worse perfomance [link to my video result](./output_images/challenge_video_output.mp4)

Here's the harder challenge video, laughably bad [link to my video result](./output_images/harder_challenge_video_output.mp4)
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The problems that I ran into with this project came from detecting lane lines due to lighting conditions, shadows, extreme curvature, or other objects in the frame. Coming up with a combination of treshold parameters that could find the lane lines on the challenge video was difficult and then updating the polyfit and where the lane lines were for the harder challenge video proved difficult also. It would be interesting to see how this detection would work in snow or with a bright lead vehicle close in front or pedestrian. 

To make my pipeline more robust you could possible update threshold parameters based on some dynamic input. 
