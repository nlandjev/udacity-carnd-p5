**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[car1]: ./examples/car1.png
[notcar1]: ./examples/notcar1.png
[car-hog]: ./examples/car-hog.png
[rectangles]: ./examples/rectangles.png
[rectangle-heatmap]: ./examples/rectangle-heatmap.png

[project_video_output]: ./output_videos/project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The extraction of HOG features is performed in the functions `extract_features` and `get_hog_features` which have been copied almost verbatim from the lectures notes. I commented out the calculation of the spatial features and the color histogram features because I wanted to start out with a simpler model - it turned out to be sufficient so I didn't add them back in. 

I started out with the parameters from the course and found out that no change was necessary:
```
colorspace = 'YUV'
orient = 9
pix_per_cell = 8
cell_per_block = 2
hog_channel = 'ALL'
```

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![Sample image of a car][car1]
![Sample image of a non-car][notcar1]

I explored a few samples and their corresponding HOG images with the above parameters. Here is one example:

![Sample of a car with corresponding HOG image][car-hog]

#### 2. Explain how you settled on your final choice of HOG parameters.

The above combination of parameters was the first I tried and seeing that the end result looked satisfactory, I saw no reason to change them. It is however very probable that the output could be improved by tweaking the parameters further.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The training of the Support Vector Machine classifier is performed in the section 'Train SVC Classifier'. I decided to use and rbf kernel initially and evaluate its performance before changing it. The test accuracy of 0.9921 was good enough to stick with the rbf kernel.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The actual work of the sliding window search is performed by the function `slide_window` which is copied directly from the lectures. The calculation of the rectangles containing cars is done in the functions `find_cars` and `find_cars_scaled`. The former is also copied from the lectures, whereas the latter performs the actual selection of window sizes and y-ranges in the image.

I settled upon the final values by estimating the rough size of a car in the different segments of the image and adjusting the scale in the respective regions. The end result could definitely be improved by adjusting these values further. My final solution uses:

```
scale=1.5 for the range (ystart, ystop) = (400, 500)
scale=2 for the range (ystart, ystop) = (400, 600)
scale=3 for the range (ystart, ystop) = (500, 600)
```

The code contains an additional value of `(ystart, ystop) = (500, 600)` for `scale = 2` which is unnecessary but which I am reluctant to remove in case I break something. The rendering of the final video took > 1hr :)

Here is an example of an image with its corresponding individual windows:

![Image with individual windows][rectangles]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I applied the algorithm described above using only HOG features in YUV color space and calculated the corresponding heatmaps. As in most cases the regions containing cars did not consist of overlapping windows I skipped the thresholding step described in the lectures.

Here is an example of a combined window and its corresponding heatmap:

![Image with combined rectangle and heatmap][rectangle-heatmap]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

Here's a [link to my video result](https://github.com/nlandjev/udacity-carnd-p5/blob/master/output_videos/project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The method for combining overlapping bounding is similar to the one described in the lecture: I created a heatmap and combined all regions contained within it. I skipped the thresholding step because each region was often detected by a single window. I then used `scipy.ndimage.measurements.label()` to construct bounding boxes to cover the area of each blob detected.

Further images of the combined windows and the corresponding heatmaps can be found in the notebook.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline is pretty basic and uses no sophisticated techniques. Some improvements might include:

* Tweaking the parameters for calcuating the HOG features
* Adding color histograms and binned color features to the feature vectors
* Using more windows of varying sizes and hot window thresholding to improve the heatmaps 
* Use information from previous frames to narrow down the search for windows in subsequent frames
