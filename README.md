# Advanced-Lane-Finding
Udacity Self-Driving Car Engineer Nanodegree Advanced Lane Finding Project. Applying computer vision to identify driving lane lines from a video stream.
---

[//]: # (Image References)


[calibr_12_drawn_corners]: ./output_images/calibr_12_drawn_corners.jpg "Chessboard Corners"
[distorted_undistorted]: ./output_images/distorted_undistorted.png "Distorted & Undistorted Cheesboard"
[processed_comp]: ./output_images/processed_comp.png "Comparative Threshold Image"
[perspective_transform]: ./output_images/perspective_transform.png "Perspective Transformation"
[histogram_lines_orig]: ./output_images/histogram_lines_orig.jpg "Starting Coordinates Histogram"
[curve_line_frames_windows]: ./output_images/curve_line_frames_windows.png "Sliding Windows Search"
[demarcated_curve_lane_lines]: ./output_images/demarcated_curve_lane_lines.png "Demarcated Lane Lines"
[inverse_transform_with_info]: ./output_images/inverse_transform_with_info.png "Inverse Transform with Radius & Offset Info"
[unmanaged_line_detection]: ./output_images/unmanaged_line_detection.gif "Unmanaged Line Detection"
[managed_line_detection]: ./output_images/managed_line_detection.gif "Managed Line Detection"



__Project Objective Result Video:__ [Project Output Video](https://www.youtube.com/watch?v=sQcXlY738_o&feature=youtu.be)



# Advanced Lane Lines Detection

The collection of documentation, codes, videos, and images in this repository,
as whole, constitute the Advance Lane Lines Detection Project, as stipulated by
the [Udacity's Self Driving Car Nano Degree](https://www.udacity.com/drive)
Program. The Project objective is to create a python program to identify the
lane lines of a road, given in a stream videos, and highlight the path where
a self-driven car can safely steer.


## Phases

This project was divided into two phases, the Exploration, and the Execution
of the pipeline.  In the Exploration Phase, we study the
images and experiment with different lane lines detection methods used later
in the pipeline to accomplish the project’s objective.  In the Execution
Phase, all the methods from the Exploration Phase are coded into a pipeline
that is feed with a stream video filmed from inside of a car looking at the
road lane from the center of it while driving.  This python program output
a mp4 video highlighting the save driving area for the car to steer.


### Exploration

There are 8 stages in the Exploration. These stages were used later as the
main building block for the main project objective.



**1.	Camera Calibration** 

To accurately read and extra dimensions from an image or video, it is necessary
to obtain the calibration parameters for the camera used to get the images or
video.  For the project, we were given chessboard images to calibration the
camera.  These images were storage in the `camera_cal` folder.  To accomplish
calibration, we read in known shapes, such a chessboard, and run couple 
function from the OpenCV library.   The first function to be used is 
`cv2.findChessboardCorners` to find the inner corners of a chessboard.
To use this function, we need to provide the function with the chessboard 
image, the inner number of columns and rows in it. After the corners have been 
identify, we use the calibrate Camera `cv2.calibrateCamera` function to obtain 
the image matrix and the distortion coefficients.

![calibr_12_drawn_corners]

**2.	Distortion Correction**

After obtaining the calibraion parameter from the Camera Calibration step, 
we need to apply distortion correction to the image before we can
take more accurate measurement information from it.  We feed the `cv2.undistort` 
function with the image matrix and the distortion coefficients obtained with 
the `cv2.calibrateCamera` function, to properly correct or undistorted the image. 
To validate that the images have been properly undistorted, we compare how the 
image look like before and after the `cv2.undistort` function has been applied, 
as show in the figure below.

![distorted_undistorted]


**3.	Color and Gradients Thresholds**

To improve lane lines detection and mitigate noise cause by light, shadows, and 
tire marks on the road, we employ Color and Gradients Thresholds on the RGB and 
the HLS color spaces.  To accomplish the desired transformation, we first create 
a copy image removing the Blue (B) color channel, in parallel another new image 
is created by removing the (L) channel of the original image.  The two resulting 
new images are merge into a binaries thresholded images denoting only lines.  The 
images below illustrate the expected output when applying of the Color and Gradients 
Thresholds.
	
![processed_comp]


**4.	Perspective Transform ("birds-eye view")**

After selecting an image for examination, I manually pick four vertices demarking the 
area of interest to later perform a perspective transform and draw a polygon to visualize 
the selected area. The destination points were chosen such that straight lanes appear 
as parallel as possible in the transformed image as shown below.  To obtained the change 
in perspective it is necessary to use the `cv2.getPerspectiveTransform` and the 
`cv2.warpPerspective` functions from the OpenCV library.

![perspective_transform]

**5.	Detect Lane Line Pixels (sliding window search)**

The process of detecting the lane line pixels is done in two steeps.  In the first 
steep, the starting points of the two lines are discover using a histogram plot.  
The histogram plots the number of pixels found along the X coordinate of the images.  
We know that one line will be to the left of the middle of the image and the other to 
the right.  The starting point coordinates are found where the Y is at it Max or bottom 
of the image and the count of pixels at it Max in that side of the image, as illustrated 
below.
	
![histogram_lines_orig]
	
In the second steep, a sliding window search is used to find and draw the lines, starting
at the coordinate found with the histogram.  After finding the x & y coordinates of 
non-zeros pixels, a polynomial of second degree is fit for these coordinates and the lane 
lines are drawn, as illustrated below.

![curve_line_frames_windows]

![demarcated_curve_lane_lines]


**6.	Lane Curvature and Off Center from the Lane**

The lane radius of curvature is calculated employing a second-degree 
polynomial fit using the coordinates of the found line pixels.  The output 
dimension is in number of pixels and latter converted to real world meters.

The mean of the lane pixels nearest to the car gives us the center of the 
lane. The center of the image gives us the position of the car. The 
difference between the 2 is the offset from the center.



**7.	Warp the Detected Lane Back onto the Original Image**

In this stage, the program performs the following steps:

* Execute an inverse perspective transform.
* Blend the processed image with the original image.



**8.	Display Lane Boundaries and Numerical Estimation**

This is the last stage, and it is where the program highlights the lane 
inner area and add the radios of curvature and offset to the original image.

![inverse_transform_with_info]


### Execution of the Pipeline

During this phase, all techniques used during the Exploration phase in addition to 
a new line searching techniques were applied to a given driving video.  The video was filmed 
looking at a road lane from the inside driving car.

This new searching techniques manage those situations where the program is not able to properly
identify a line in the current video frame.  In such circumstances, the program extrapolates 
from the average line data point from the last previous three frames.  The resulting out videos
can be seen below.


---

## Discussion

### Gradient & Color Thresholding

Detecting lines taking in days too bright or too cloudy can be a challenge.
During the Exploration phase, we learned that using Gradient on the X axis,
removing the Blue channel from the image and reducing the L channel in Color Space
can improve lines detection.

### Managing Bad Frames

Below is a list of circumstances that could make line detection a challenge even after 
applying Gradient and Color Thresholding techniques.  

* Too bright or too cloudy days
* Shadows on the road
* Car goes into a tunnel
* Sharper turns
* Tire or other marks on the road


The following two short animations illustrate the improvement in line detection when having a manage line detection approach.   The challenge in the original video involve a sudden change in the lane color, asphalt road to concrete bridge, and some tire marks.  Even though the manage line detection video is not perfect in this case, it is significantly better than the unmanage approaches, I believe that it is just matter of tuning how many frames to average. 

![unmanaged_line_detection]
![managed_line_detection]

The performance of the pipeline used in this project did not work to expectation on the given challenge.  The result is completely unacceptable, see harder_challenge_video.mp4 videos on the `output_images` folder.
