# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goal of this project is to build a simple pipeline that finds lane lines on the road. The pipeline uses basic Computer Vision algorithms to identify lanes on the road. There are potential shortcomings with the current pipeline, which are identified, and methods to address them are also elaborated.

[image1]: ./test_images/solidWhiteCurve.jpg 
[image2]: ./test_pipeline_results/color_selected.png "Grayscale"
[image3]: ./test_pipeline_results/region_selected.png "Grayscale"
[image4]: ./test_pipeline_results/gauss_filtered.png "Grayscale"
[image5]: ./test_pipeline_results/canny_detected.png "Grayscale"
[image6]: ./test_pipeline_results/hough_results.png "Grayscale"
[image7]: ./test_pipeline_results/output_extrapolated.png
---

### Reflection

### Summary of the pipeline

The pipeline consists of six steps to find lanes on a single image.

The steps are enumerated below.

* Step 1: Select the colors of interest.
* Step 2: Select the region of interest.
* Step 3: Apply gaussian filter to smoothen the images.
* Step 4: Find edges using canny edge detector.
* Step 5: Create lines from edge pixels using Hough transform.
* Step 6: Superimpose the extrapolated lines on original image.

### The original image

The original image on which the pipeline is applied is shown below.

![alt text][image1]



### Step 1: Select the colors of interest.

As the first step, the colors of interest are selected. In our case, since the lanes are either white or yellow, I have selected them as colors of interest. 

The efficiency of color selection algorithm greatly depends on the color space of the input images. Therefore, the images are converted from RGB to HSV (Hue, Saturation, Value) color space. The maximum and minimum thresholds for filtering our colors of interest are identified and the corresponding colors are filtered using `cv2.inRange()` function.

The output from this pipeline is

![alt text][image2]


### Step 2: Select the region of interest.

As the next step, the region of interest is identified from the images. The vertices are represented relative to the shape of the image. The helper function `region_of_interest()` is used to filter out the region of interest.

The output from this pipeline is

![alt text][image3]


### Step 3: Apply gaussian filter to smoothen the images.

As the next step, gausssian filter is applied to smoothen the images. Here the helper function `gaussian_blur()` is used with a kernel size of 5.

The output from this pipeline is

![alt text][image4]


### Step 4: Find edges using canny edge detector.

From the Gaussian blurred image, the edges are detected using the helper function `canny()`. Through trial and error, the maximum and minimum threshold values, which gives optimal performance (i.e.) that finds the maximum edges, have been fixed as 30 and 100 respectively.

The output from this pipeline is

![alt text][image5]


### Step 5: Create lines from edge pixels using Hough transform.

As the next step, from the edge pixels lines are constructed using the Hough Transfer algorithm. This is accomplished by calling the helper function `hough_lines()` with parameters `rho=1, theta=np.pi/180, threshold=30, min_line_len=5, max_line_gap=10`.

Inside the `hough_lines()` function there is a `draw_lines()` function which creates lines on the image using end points from Hough transform.

The main challenge in this project is to extrapolate/extend broken lines as a continuous lane line. For this purpose, the `extrapolate_lines()` function is used. 

The `extrapolate_lines()` consists of two main functions.

1. `classify_lanes()` -- which classifies the end points from Hough transform to either right or left lane.
2. `sketch_lanes()` -- which draws the extrapolated line onto the image.

#### Classification of lanes

The function `classify_lanes()` classifies the end points based on its slope values. The slope of a given line segment is calculated using two point formula. The two points here are the end points from Hough transform. Since the lanes represent the sides of a triangle, they can be classified using their sign/direction (i.e. positive or negative slope). The threshold for classification is fixed at 0.3 in order to exclude edge-lines that do not represent the lanes.

#### Sketching the lanes

After classifying the lanes, the points from the one lane are fit into a line using the `numpy.polyfit()` function. In order to extrapolate the line, the x-intercept of the line with the two extremes of y lines are calculated. Finally, a line is drawn on the image using the `cv2.polylines()` function. 

In addition to the above functionality, a logic to reduce the flickering effect on videos is also added. The flicker is caused due to the rapid change in slope and y-intercept (i.e. line equation) of the drawn line on a particular image/frame. In order to smoothen the effect, the mean value of the line equation from last N images/frame is calculated. Therefore any peak value is smoothened out. The larger the N buffer the smoother the transition is, but less reactive to changes in lanes (particularly visible on curves). 

This logic also has an additional advantage that when there are no edges detected in a particular frame the mean value is used to make an approximation on the particular image/frame.

The output from this pipeline is

![alt text][image6]


### Step 6: Superimpose the extrapolated lines on original image.

As the last step, the image containing the lane lines are superimposed on the original image. 

The final output is 

![alt text][image1]


### 2. Shortcomings with the current pipeline

#### Shortcoming identified and resolved

The following shortcomings have been identified and resolved.

* In the challenge video, several edges were identified, when a simple grayscale conversion was used in Step 1. In order to resolve this, color selection was used as a pre-processing step to detect the lane lines instead of relying solely on canny edge detector to identify lanes from grayscale image.

* There was a visible flickering effect when the pipeline was used on series of frames aka video. In order to reduce this, the history of previous frames was used to smoothen out outliers.

* Also in the challenge video, there were some frames where lane lines couldn't be identified and the pipeline failed throwing an expection. The above method also solved this issue and an approximate line derived from history is drawn on the image.

#### Shortcoming identified and unresolved

The following shortcomings have been identified with the current pipeline:

* In case of visible curves, the approximation for straight lines fails miserably.

* In case of changes in color of lane lines due to shadows and lighting effects, this algorithm is not robust enough.

* The region of interest and other threshold values have been finetuned only for the given set of images and videos. There is a possiblity that these values could either be entirely wrong or be finetuned for another set of scenarios.


### 3. Possible improvements to the pipeline

* In order to mark curved lames, higher order polynomials could be used instead of line (a polynomial of order 1). 

* Another possibily is to increase the frames per second (fps) output from the camera and use the approximation from a high N number of frames.


