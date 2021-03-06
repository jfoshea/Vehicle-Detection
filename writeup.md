# Self Driving Car - Vehicle Detection 

## Overview
The goals / steps of this project are the following:

- Extract vehicles, non-vehicles and test data and visualize a small sample of images
- Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images
- Train a classifier Linear SVM classifier
- Normalize the features and randomize a selection for training and testing.
- Implement a sliding-window technique and use the trained classifier to search for vehicles in images.
- Create bounding boxes for vehicles detected.
- Visualize boxed and heatmap images
- Create a pipeline to video stream generation
- Run the pipeline on the test_video.mp4 and the full project_video.mp4.

The jupyter notebook can be found here: [VehicleDetection.ipynb](https://github.com/jfoshea/Vehicle-Detection/blob/master/VehicleDetection.ipynb)

### Training Data
The training data was supplied as two zip files. I extacted the vehicle and non-vehicle into the local git repository and I extracted both sets of images into two lists within the jupyter notebook. There were 8972 images in vehicles and 8968 images in non-vehicles. I then plotted a random image from each data set as seen in cell 3.

### Histogram of Oriented Gradients (HOG)
A feature descriptor is a representation of an image or an image patch that simplifies the image by extracting useful information and throwing away extraneous information. The feature vector is very useful for object detection like cars on the road, and in this project the feature vector is fed into a Support Vector Machine (SVM)  classification algorithm to detect cars. To calculate a HOG descriptor, we need to first calculate the horizontal and vertical gradients in order to determine a histogram of gradients. Luckily the algorithm is available via scikit-image and OpenCV with APIs to allow us to implement to compute and extract HOG features from a data set. I read somewhee that the scikit-image HOG was better cv2.HOGDescriptor as it had more options but I cannot find the reference to that article now. I initially coded up `get_hog_features` using `scikit-image HOG`, and stuck with paramerters given by the example: `pix_per_cell 8`, `cell_per_block 2`, `orient 9`. I then experimented with different color spaces and I noticed the number of features detected varied when switching between RGB, YUV, HSV, but I am not sure if is determined by color_space alone. I then tried YCrCb and noticed an overall improvement as the number of features detected increased. I then tried changing orient pix_per_cell, cell_per_block, and often times the application would crash either during feature detection or training. I then tried the OpenCV HOG feature detector, and I found that cv2 version worked well for this project and it actually ran faster detecting features, especially when utilizing a search within an existing HOG descriptor rather than computing the descriptor over and over.Here are some articles I read cv2.HOGDescriptor: [HOG blog post](https://www.learnopencv.com/histogram-of-oriented-gradients/) and [Stackoverflow HOG post](https://stackoverflow.com/questions/6090399/get-hog-image-features-from-opencv-python). The final config config was to use YCrCb color space using OpenCV HOG on all channels. I found that `orient 11` allowed the feature detection to work well and not to crash as often and I also used `pixels_per_cell 8`, `cells_per_block 2`. Please see cell 2, and 6 that define the parameters and the feature extraction calls to vehicles and non-vehicles. The final settings detects 9,540 features in each data set (see cell 6). Note: There are so many parameters passed around between functions that I decided to package them into a dictionary called `params` and pass the params to function calls as it cleans up the code and makes it more readable (at least to me).

### Classification Algorithm
As mentioned previously the feature vectors are fed into a classifier to detect cars. There are two data set vehicles and non-vehicles. Before running the classification algorithm I normalized the extracted features from each data set using the `StandardScaler from sklearn.preprocessing`, see cells( 7-9 ). I then used the `sklearn.model_selection train_test_split` to split that data sets into training and test data using an 80:20 split. It was suggested that a Support Vector Machine (SVM) would be a good choice for this project and this is the classification algorithm used to detect cars. The code to run the SVM can be found in cells 10, 11. The accuracy of the SVM was quite high 99.24% so I stuck with this classification technique. 

### Sliding Window Search
The supplied test_images folder contains six images. I first draw_boxes routine using hard-coded boxes to visually see how the boxes might look, see cell 13. All of the hard coded boxes were in the lower half of the images, so a sliding window search should be contained here to speed up processing and reduce the probability of false positives being detected. To detect cars with different perspective views a multi-scale window search is needed. I started with xy_window = (96, 96) and 0.5 for overlap as suggested in the example code.  For multi-scale windows and perspective views I initially had three different different window scales (64, 96,128) and I had mixed success with bounding boxes at different perspectives, I then took inspiration from this implementation and increased the number of multiscale windows to five ( 80, 112, 128 and 160), and changed the overlap to 0.6 [github](https://github.com/hfoffani/Vehicle-Detection) I found that changing the percent overlap can change the results significantly, and when I changed the percent overlap to 0.6 this reduced the number of hot_windows and more efficiently detect cars. To restrict the search window to view of the road the `y_start_stop = [ 400, 600 ]` was selected to compute windows more efficiently.

### Heatmap and Bounding Boxes
The sliding window and perspective_windows search identified the positions of positive detections and appended them to hot_windows for each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  Please see cell 15 which shows the pipeline visualization for all test images. 
An example of for test6.jpg is also shown here: ![Alt text](writeup_images/pipeline_stages.png "test image 6 pipeline stages")

### Video Implementation
The output videos can be found here: 

[output_test_video.mp4] (https://github.com/jfoshea/Vehicle-Detection/blob/master/output_test_video.mp4)
[output_project_video.mp4] (https://github.com/jfoshea/Vehicle-Detection/blob/master/output_project_video.mp4)

### Conclusion
This was a very interesting project. I spent a lot of time experimenting with trying different color spaces and HOG parameter selection in order to classify images with high accuracy. As mentioned in the writeup I ended up selecting YCrCb as the color space of choice. Testing the slide windows and perspective windows in paricular were also time consuming. In many instances I got many false detections and ended up with bounding boxes all over the screen. By changing the percent overlap and searching using multiple perspective window scales helped focus in on largely positive detections, but searching using multi-scale windows increases the compute time. I noticed that most of the images are in the daytime, it would be intersting to see if the pipeline would work or fail miserably in low light conditions. I also noticed the data set consisted mainly of cars, I didnt notice any trucks or busses in the data set when randonly running the car feature visualization cell.
One other thing I defintely got frustrated with was the ipython kernel crashing a lot for this project, I didnt see this before on other projects. I also had problems with VideoFileClip for this project, I think my ffmpeg got in a bad state. I tried installing ffmpeg again but it didnt help. I ended creating conda enviornment on another machine in order to get VideoFileClip to generate videos. 

