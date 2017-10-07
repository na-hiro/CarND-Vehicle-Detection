# Vehicle Detection
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


The Project
---

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector.
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

Here are links to the labeled data for [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) examples to train your classifier.  These example images come from a combination of the [GTI vehicle image database](http://www.gti.ssr.upm.es/data/Vehicle_database.html), the [KITTI vision benchmark suite](http://www.cvlibs.net/datasets/kitti/), and examples extracted from the project video itself.   You are welcome and encouraged to take advantage of the recently released [Udacity labeled dataset](https://github.com/udacity/self-driving-car/tree/master/annotations) to augment your training data.  

Some example images for testing your pipeline on single frames are located in the `test_images` folder.  To help the reviewer examine your work, please save examples of the output from each stage of your pipeline in the folder called `ouput_images`, and include them in your writeup for the project by describing what each image shows.    The video called `project_video.mp4` is the video your pipeline should work well on.  

---
[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]:./output_images/HOG_example.png
[image3]: ./output_images/sliding_windows.png
[image4]: ./output_images/bboxes_and_heat.png
[image5]: ./output_images/multi_sliding_window.png
[video1]: ./output_test_video.mp4
[video2]: ./output_project_video.mp4
[video3]: ./output_ld_project_video.mp4


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### README

### Histogram of Oriented Gradients (HOG)

#### 1. Extraction of HOG feature from training image

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

#### 2. Input data of HOG feature quantity

As the HOG feature quantity, it is necessary to decide what kind of data to input. As the image format, there are expressions in various color spaces such as RGB, HLS, YUV and the like. I tried all methods, but I decided to use YCbCr image as input of HOG feature quantity.

![alt text][image2]

#### 3. Final selection of HOG parameters
I trained a linear SVM by using training data.
The set value of the hog feature is as follows.
* orientations: Number of bins in histogram (9)
*  pixels_per_cell: Cell size (8, 8)
* cells_per_block: Number of cells per block (2, 2)

As shown in the figure below, similar to cars and cars, the gradient direction of the cell brightness can be sufficiently discerned by visual observation and the actual test results also gave good results.

![alt text][image3]

### Sliding Window Search

#### 1. Sliding window behavior

This section shows an example of a sliding window.

The scaling factor is 1.3. Thus, while scale image to 1 / 1.3, while moving by two cells, to calculate the HOG features, normalize, and identified using the trained model.

If it is identified as a vehicle, it is voted as a candidate box and the score is accumulated. We will use this integrated data as a heat map. The heat map is the result of detecting a vehicle in the image, but the larger integration value indicates that the accuracy is higher. Therefore, set the threshold, carefully select the vehicle candidates from the heat map, label them as one vehicle, and draw with a rectangle.

Below, from the left, original image, heat map, thresholded heat map, final result are shown.

As an example, it is shown that vehicles are more carefully selected from vehicle candidates.

![alt text][image4]

#### 1. 2 Multi-Scale Sliding window

Detection using sliding windows of a plurality of sizes is described below.

In the example in the previous section, we have used a single scale to detect, but there are ways of using windows of multiple sizes to make detection more accurately and efficiently.

For example, when the vehicle to be detected is far and near, the size of the vehicle is different. Of course, if the vehicle is located far away, the scaling value should be small. Conversely, in the case of a vehicle located nearby, the larger the scaling value, the more efficient detection can be made. That means you can omit unnecessary calculations.　The Window range specification and scaled multi window are defined as follows. The image used is different in the position of the vehicle. Good results were obtained with the setting as below.

![alt text][image5]
---

### Video Implementation

#### 1. Pipeline Process
In the previous section, processing of a single still image was explained.In this section, we describe the functions added to motion picture processing.If the single sheet processing result is displayed as it is, the detection result is not stabilized. Therefore, hold the heat map of multiple frames (15 frames in this project) as internal information and calculate the average value. The result obtained is the heat map before threshold processing. After that, threshold processing and labeling make it possible to obtain stable and robust detection results.


* Here's a [link to my video result](./output_project_video.mp4)

* Here's a [link to my test video result](./output_test_video.mp4)

* Here's a [link to Video resulting from integration with advanced lane detection](./ld_project_video.mp4)

---

### Discussion

The effectiveness of vehicle detection using HOG and SVM included in conventional machine learning was shown this project. However, the following problems can be mentioned.
* BBox is displayed besides the vehicle. False detection occurs.
* Detection processing time is long. In a general-purpose LSI for automotive use, real-time processing is difficult.
* The fitness degree of the BBox and the vehicle may be low. When the distance meter is used only with the monocular camera, the higher the fit degree, the more accurate distance calculation becomes possible.
* In some cases, it can not be detected normally due to occlusion.

To improve detection accuracy, vehicle detection using R - CNN on the GPU is effective. For example, Yolo, SSD, Multi-net, etc. This is considered to make it possible to detect robust against occlusion, directional differences, etc. When using conventional machine learning, I think that images with different sizes can be detected in a short time by using pyramid images in addition to the search range applied this project.
