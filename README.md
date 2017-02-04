## Vehicle Detection

**Project Description**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./images/car_notcar.png
[image2]: ./images/HOG_features_HLS.png
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

### Please see the [rubric](https://review.udacity.com/#!/rubrics/513/view) points

---
### Data Exploration
Labeled images were taken from the GTI vehicle image database [GTI](http://www.gti.ssr.upm.es/data/Vehicle_database.html), the [KITTI](http://www.cvlibs.net/datasets/kitti/) 
vision benchmark suite, and examples extracted from the project video itself. All images are 64x64 pixels. 
A third [data set](https://github.com/udacity/self-driving-car/tree/master/annotations) released by Udacity was not used here. 
In total there are 8792 images of vehicles and 9666 images of non vehicles. 
Thus the data is slightly unbalanced with about 10% more non vehicle images than vehicle images.
Images of the GTI data set are taken from video sequences which needed
to be addressed in the separation into training and test set.
Shown below is an example of each class (vehicle, non-vehicle) of the data set. The data set is explored in the notebook `exploration.ipynb` 

![sample][image1]


### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

Due to the temporal correlation in the video sequences, the training set was divided as follows: the first 70% of any folder containing images 
was assigned to be the training set, the next 20% the validation set and the last 10% the test set. In the process of generating HOG features 
all training, validation and test images were normalized together and subsequently split again into training, test and validation set. Each set was shuffled individually. The code for this step is contained in the first six cells of the IPython notebook `HOG_Classify.ipynb`. I explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  
I selected a few images from each of the two classes and displayed them to see  what the `skimage.hog()` output looks like. Here is an example using the `HLS` color space and HOG parameters of `orient=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![HOGchannels][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I went through a number of different combinations of color spaces and HOG parameters and trained  a linear SVM using different combinations of HOG features extracted 
from the color channels. For HLS color space the L-channel appears to be most important, followed by the S channel. I discarded RGB color space, 
for its undesirable properties under changing light conditions. YUV and YCrCb also provided good results, but proved to be unstable when all channels were used. 
Ther was relatively little variation in the final accuracy when running the SVM with some of the individual channels of HSV,HLS and LUV. 
I finally settled with HLS space and a low value of `pixels_per_cell=(8,8)`. Using larger values of than `orient=9` did not have a striking effect and only increased the feature vector. 
Similarly, using values larger than `cells_per_block=(2,2)` did not improve results, which is why these values were chosen. 

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using all channels of images converted to HLS space. I included spatial features color features as well as all three HLS channels. 
The final feature vector has a length of 5292, most of which are HOG features. When trained on the training set this resulted in a validation accuracy of 96.4% and a test accuracy of 97.3%. The average time for a prediction (average over a hundred predictions) turned out to be about 5ms on an I7 processor, 
thus allowing a theoretical bandwidth of  200Hz. A realtime application is therfore only feasible
if several parts of the image are examined in parallel. The sliding window search  described below is an embarrassingly parallel task and corresponding speedups are expected, 
but implementing it is beyond the scope of this project.   Using just the L channel reduced the feature vector to about a third, while  test and validation accuracy dropped to about 94.5% each.
Unfortunately, the average time for a prediction remained about 5ms. 
The classifier used was `LinearSVC` taken from the ´scikit-learn´ package. 

###Sliding Window Search
####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions at random scales all over the image and came up with this (ok just kidding I didn't actually ;):

![alt text][image3]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to try to minimize false positives and reliably detect cars?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

