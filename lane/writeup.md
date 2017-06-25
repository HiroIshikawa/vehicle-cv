##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./output_images/test1.png "Road Transformed"
[image3]: ./output_images/binary_combo_example.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image6]: ./output_images/example_output.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 5-7th code cells of the IPython notebook.  

First of all, I prepared "object points", which represents 3d coordinates of the chessboard corners in real world space. `objp` is the variable to store the each coordinates of the points in the 9x6 grid without considering depth using meshgrid with numpy functions. For each provided camera calibration images, I applied the cv2.findChessboardCorners() on the grayscaled image for 9x6 corners. If any chessboard having 9x6 corners detected, it stores a copy of`objp` defined above to `objectpoints` which is a list for the real world coordinatse and found corners information to `imgpoints` which preresents the list of coordinates of detectd corners in the images. 

With computing the `img_size` of the image to be calibrated, in addition to the `objpoints` and `imgpoints`, I applied `cv2.calibrateCamera` to extract required distortion coefficients.

Finally, I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for thresholded binary image pipeline is summarized in the 20th cell in the ipython notebook.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines 6 through 33 in the 93th cells. From the exploration of different color channels for applying thresholiding, I found L channel seems to have clearler color contrast around the target lines to extract gradient features. For the direction of the gradient, since the target lane lines should be aligned in vertical from the perspective of camera, I chose x-sobel, which find gradient in x direction, to capture the lane line features. For the color thresholding, S channels seems to have the most contrast around the target line compared to other color challens (Grayscaled, RGB, or other two (H,L) spaces in HLS). I combined these two thrading by tuning each thresholding values to accomplish the best result for the test image.

Here's an example of my output for this step.

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a preconfiguration and function to perform the transform, which appears in the 21th cell in the notebook. The `perspective_transform()` function apply the transform with given `binary` image generated in the previous step and source and distination coordinates. I chose the source and destination points in the following manner:

```
src = np.float32(
    [[center_x+60,center_y+80],
     [img_size[0]*(8/9),img_size[1]],
     [img_size[0]*(1/5),img_size[1]],
     [center_x,center_y+80]])
dst = np.float32([
    [img_size[0]*(3/4),0], 
    [img_size[0]*(3/4),img_size[1]],   
    [img_size[0]*(1/4),img_size[1]], 
    [img_size[0]*(1/4),0]])
```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 700, 400      | 960, 0        | 
| 1138, 720     | 960, 720      |
| 256, 720      | 320, 720      |
| 640, 440      | 320, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then, on the warped binary image, I made the lane lines finding using peaks in histogram, sliding windows, and polynomial fit, which appears in the 22nd cell in the notebook. I computer the sum of pixel values for each column of the half bottom of the binary image to take histogram. To find the left lane line, I took the column having maximum value from the first half using `np.argmax()`. For the right lane line, I did the same to the second half. To efficiently search the rest of the lane line's segements, I used the sliding window approach. I tuned the width margin of the window. Based on the extracted line lanes positions from the sliding windows, I calculated the polinomial with `np.polyfit()`.

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

The calculation of radius of curvature is included in the 25th cell in the notebook. 

To compute it, first of all, I defined the conversions in x and y from pixels space to meters given measurement data. With that, recomputed the polynomials with the conversion rate, which is the variable `left_fit_cr`/`right_fit_cr` in the equation for radius of curvature below.

`left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Please watch the video file called "project.mp4" attached in the folder.

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline to define the two lane lines might be failed if there's multiple vertical lines other than the target lane lines. The color channel and x-sobel gradient based binary image convertion might not be enough to elimiante these cases. In that case, I should explore different approaches to define the lane lines like incorporating fine tuning of magnitude or direction of the gradient. Also, when applying perspective transform, redefining the parameteres for the source points may help to define finer lane lines. Since I am using the detected coordinates of lane lines in the previous frames, without introducing resetting the base lane line coordinates for the reference for any abnormal detection, it might start generating corrupted lane findings for successive frames. As a remedy for this, I may introduce a logic to switch the lane line detection between the one with using previous lane findings and the original histogram based lane line detection. 
