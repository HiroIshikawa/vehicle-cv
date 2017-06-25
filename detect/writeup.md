##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]: ./output_images/RGB.png
[image3]: ./output_images/LUV.png
[image4]: ./output_images/YUV.png
[image5]: ./output_images/HSV.png
[image6]: ./output_images/window_example.png
[image7]: ./output_images/pipeline_ex1.png
[image8]: ./output_images/pipeline_ex2.png
[image9]: ./output_images/heat1.png
[image10]: ./output_images/heat2.png
[image11]: ./output_images/heat3.png
[image12]: ./output_images/heat.png
[image13]: ./output_images/heat_applied.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 1-7th code cell in the jupyter notebook.  

I started by reading in all the `vehicle` and `non-vehicle` image.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB`,`LUV`,`YUV`,`HSV` color spaces and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

RGB

![alt text][image2]

LUV

![alt text][image3]

YUV

![alt text][image4]

HSV

![alt text][image5]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and settled in this configuration below:

`
cspace='YUV'
spatial_size=(32,32)
hist_bins=64
hist_range=(0, 256)
orient = 12
pix_per_cell = 8
cell_per_block = 2
`

The reason why I selected YUV as the color space was because it was the only color space that avoided false positives appeared in some frames with shadows in the project video. The spatial size is chosen because it seems to be able to retain significant features of cars while reducing computational cost compared to higher resolutions. I used the relatively higher histogram bins values to add more features. Histogram value range is retained to default. Increasing orient value helped the features can be more variant so that it can be distinguished accurately car particular features from non-car features. I reduced pixel per cell value to examine smaller size of region and more features to extract from it. Following the rule of the thumb, I normalized the gradients for a block consisted of two cells. 


####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The code for this step is contained in the 8-18th code cell in the jupyter notebook.  

I started preparation for training from stacking the all features given by spatial bins, color histograms and HOG feature extractions. Then I applied standard scaling to it using `sklearn.preprocessing.StandardScaler()` and splitting the data into randomized training and test sets with `sklearn.model_selection.train_test_split()`. I tried to figure out the best C value using grid search and it found the default C=1.0 still best so left that value for the training. Finally, I trained the model with `sklearn.SVM.LinearSVC()`.

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The code for this step is contained in the 19-20th code cell in the jupyter notebook. 

I decided to search window positions at a fixed scale 1.15 with 64(8x8) sampling rate and 1 cell per step under the same cell/block configuration for the HOG feature extraction above (pix_per_cell=8, cell_per_block=2). I implemented the dynamic sizes of the window as the slide goes down to the bottom of the frame but this did not perform better than the configuration above. The step size of the window is set and this made 87.5% overlap between pair of windows. This caught features more likely than larger value of the step size.

The image below is one of the example with the windows configured:
![alt text][image6]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I searched on single fixed scale using YUV 3-channel HOG features plus spatially binned color and histograms of color in the feature vector with cutting off classification results having low confidence using `sklearn.SVM.LinearSVC.decision_function()`.  The cutoff with confidence value was found effective to reduce false positives. Here are some example images:

![alt text][image7]
![alt text][image8]

---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

Please check out the "project_video.mp4" in this zip for the video implementation.


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I kept the coordinates of positive detection in each frame of the video. From the positive detections, I accumulated these detections over six succesive frames and created a heatmap using the detections. Then, I thresholded that map to identify vehicle detection. Finally, I made and put rectangles defined through the procedure above over the processing image.


### Here are six frames and their corresponding heatmaps:

![alt text][image9]
![alt text][image10]
![alt text][image11]

### Here is the output of the integrated heatmap from all six frames:
![alt text][image12]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image13]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I found false positives in my final implementation on the video. Without improving the classifier itself, it can be eliminated through tuning the decision funciton thresholding. However, this may cause overcompensation of detecting features. Also, the heat map thresholding may be able to solve this problem. But this may cause same issue as decision function thresholding too. Finally, the improvement of the classifier training itself is the promising idea and especially the hard negative sampling sounds effective way to solve the false positives issue without overcompensating accuracy.