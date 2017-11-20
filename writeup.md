## Writeup Template
### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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
[image1]: ./example_imgs/HOG_img.png
[image2]: ./example_imgs/sliding_windows_test_imgs.png
[image3]: ./example_imgs/HOT_pre_filter.png
[image4]: ./example_imgs/HOT_5.png
[image5]: ./example_imgs/Window_5.png
[video1]: ./project_video.mp4
[result]: ./project_video_vehicle_detection.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.
In the first cell of the python notebook, it can be seen that the cars and notcars data is loaded, and the size of each dataset is displayed. For clarification, ther are 8792 car images, and 8968 notcar images. Cell 2 consists of all the necessary imports required for the project, and cell 3 consists of most of the functions that are required. In cell 3, functions for getting and extracting the HOG features, computing binned color features, computing the color histogram features and the sliding and searching window functions used searching images for cars as well as the draw boxes function to draw boxes onto the image where the cars are located. Finally, Cell 4 uses these functions to extract the car and notcar features, and displays a comparison showing a random sample of each, as well as their respective HOG image, which can also be seen below.

![alt text][image1]

#### 2. Explain how you settled on your final choice of HOG parameters.
Parameter tuning was mostly done in cell 5.

I chose the YCrCb color space since this performed somewhat better than the RGB color space by displaying less false positives. Nine HOG orientations were used due to the fact that using more than 9 leads to great increases in computation time with marginal gain in actual performance. Also, I chose to use all HOG channels, since this provided me with the entire color spectrum, and the pipeline would have some difficulty finding cars at times. 

By only using parts of the image, computation time and performance was drastically increased. This was done by ignoring the upper half of the image, since tree tops and the sky are not of interest for detecting cars, unless we're looking at flying cars. Also, a small portion of the bottom bit of the image was also ignored, since the hood of the car was not of interest either. Worth mentioning here is that a fair portion of the left side of the image was also ignored, due to the fact that the vehicle was driving then the lane to the far left, an no vehicles of interest were present on the left side. However, this would not be appropriate if this pipeline was to be used in general, since a vehicle is not always located on the lane to the far left. But for this assignment, this was an improvement.


With regards to pixels per cell, cell per block, no experimentation was done here. Finally, the values 16 and 32 were tested for spatial size and hist bins. 

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

My calssifier was trained in cell 6, using linear support vector classification (linear SVC). A total of 10% of the data was used for testing the accuracy of the classification, which reached a wooping 99.32%

### Sliding Window Search

#### Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows? Also show some examples of test images to demonstrate how the pipeline is working.

Cell 7 contains a function called find_car, which performs a sliding window searh and returns the windows and the image. Cell 8 shows an example of the function's output, which is also displayed here below.

![alt text][image2]

Here, six test images were used to determine the performance during the initial parameter tuning for this function. It can be seen that some false positives occur. I tried varying the window size between 64, 96 and 128. 96 seemed to miss the least vehicles, while providing the least false positives. I also tried varying the overlap, beginning with 0.5, and increasing by increments of 0.05 until an increase in accuracy was seen. Larger overlap did not provide much value for this window size in this initial parameter tuning without greatly enhancing computation time.

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_vehicle_detection.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

If you look at the 6 test iamges above, you will find a false positive in the top left image. To filter out such false positives, I began with generating a heatmap. Here, a black image is first generated in the same size as the image being analyzed, and a value of +1 is added to each pixel where a window is located. More windows = hotter area. The idea behind this method is to isolate one-time images that would be wrongly classified, and a heatmap of the image with a false positive is provided below.

![alt text][image3]

A threshold of 2 was chosen, which means a total of 3 boxes or more must be present to determine whether a car was located or not. The result of this filter (both heatmap and the final result) can be seen below.

![alt text][image4] 


Cell 14 contains a function called drawl_labeled_bboxes, which is used to draw one larger box instead of multiple boxes. The output of this function can be seen below.

![alt text][image5]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

Using this pipeline on the entire video providid initial satisfacory results only, since unacceptable performance for a certain series of frames was observed. In order to get around this, I decided to perform two different sliding window searches. One with a larger window size and smaller overlap, and one with a smaller window size and larger overlap. This in turn lead much more false positives, which were mostly filtered out using the threshold of 2. While this change provided a satisfactory result in accuracy, computation time increase drastically, resulting in almost two hours for processing the video. 

One way of decreasing this computation time is by only performing certain scans in certain areas of the image. For example, using smaller windows for cars that are further away (higher up in the image), and larger windows for cars that are closer (lower down in the image). Another way of improving the pipeline would be to increase accuracy by reducing the dependency on multiple scans. This can be done by initailly locating a car, estimating its trajectory, and using this estimate to determine where the car is located in the image.
