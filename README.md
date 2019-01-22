# **Finding Lane Lines on the Road**

[//]: # (Image References)

[image1]: ./test_images/solidWhiteCurve.jpg "Original Image"
[image2]: ./docs/solidWhiteCurve_1.jpg "Colour Selection"
[image3]: ./docs/solidWhiteCurve_2.jpg "Grayscale Conversion"
[image4]: ./docs/solidWhiteCurve_3.jpg "Gaussian Blur"
[image5]: ./docs/solidWhiteCurve_4.jpg "Canny Edge Detection"
[image6]: ./docs/solidWhiteCurve_5.jpg "Region of Interest Crop"
[image7]: ./docs/solidWhiteCurve_6.jpg "Hough Lines"
[image8]: ./docs/solidWhiteCurve_7.jpg "Line Combination"
[image9]: ./docs/solidWhiteCurve_8.jpg "Image Combination"

---

## Pipeline Architecture

This pipeline consists of 8 steps:

1. Colour Selection
2. Grayscale Conversion
3. Gaussian Blur
4. Edge Detection with Canny Transform
5. Region of Interest Crop
6. Hough Lines
7. Line Combination
8. Image Combination

Each step is described in the following, and an example is shown with the following image:

![alt text][image1]

#### Colour Selection
The first step is to select only the yellow and white pixels for lane detection. This is achieved in the following procedure using OpenCV functions:

1. Set an upper and lower HLS threshold for the yellow and white colours
2. Convert the image to HLS
3. Filter the images using the thresholds to create an image with only yellow pixels and an image with only white pixels
4. Combine the white and yellow images
5. Use the combined image as a mask to select only the yellow and white pixels in the original RGB image.

HSL is used since this colour space makes it easier to isolate yellow and white from the rest of the image. The output of this step looks like this:

![alt text][image2]

#### Grayscale Conversion
The next step is to do a grayscale conversion, which is done with a simple OpenCV conversion. This is done to simplify the further procedure as colour is no longer needed for edge detection. The output of this step looks like this:

![alt text][image3]

#### Gaussian Blur
Next, the Gaussian blur is performed to reduce the effects of noise, which will make edge detection much more difficult. This is also achieved with a simple OpenCV function. Different kernel sizes were experimented, and it is found that a size of 3 achieved the best results.The output of this step looks like this:

![alt text][image4]


#### Canny Edge Detection
After, the Canny edge detection was accomplished with a simple OpenCV function. The Canny edge detection requires and upper and lower threshold as parameters, where the upper threshold sets the gradient limit for strong edges that will be included in the edge detection. Any gradient below the lower threshold is discarded as an edge, and gradients between the threshold will be included if it is connected to a strong edge. Recommended high:low thresholds rations are 3:1 or 2:1, and it was experimentally found that a lower threshold of 100 and a high threshold of 300 achieved good results. The output of the Canny edge detection is a black image with white pixels along the edges.The output of this step looks like this:

![alt text][image5]


#### Region of Interest Crop
The edge detection algorithm outputs all edges in an image, but it is desired to only have the edges that correspond to lane lines. Therefore, a region of interest in image space can be defined where the lanes will occur inside. This can be formed using trapezoid connecting the bottom of the screen to about halfway up the screen with edges parallel to the lane lines. This will discard all edges from adjacent lanes, cars, etc.The output of this step looks like this:

![alt text][image6]


#### Hough Lines
Next the edges need to have line segments fit through them. This is accomplished using a hough line transform, which looks at every point along the edge and finds the every possible line that fits through that point. These lines are plotted in hough space as polar coordinate parameters, and this is done for each point along the edge. The intersections in hough space correspond to lines that cross multiple points, which is likely to be a line segment that fits the edge. This is done using a voting system, and the output of this is multiple line segments along the edges.

The threshold for number of intersections to detect a line is set as 5, and the minimum number of points that can form a line is set as 5 as well. Lastly the maximum gap between two points to be considered in the same line is set as 2. The output of this step looks like this:

![alt text][image7]


#### Line Combination
After getting line segments, these need to be combined to form one line for the left lane and one for the right. The first step is to classify each line using the slope. Slope values that are small are discarded (since they likely do not correspond to a lane), while negative slope values correspond to left lanes and positive slope values correspond to right lanes due to the coordinate system that OpenCV uses. Once classified, the start and end points of each line segment are used as points, and a least squares linear regression method is used to fit a line to the left points and the right points to create a line for the each lane. The output of this is an image with the two lanes drawn. The output of this step looks like this:

![alt text][image8]


#### Image Combination
The original image and the image with the two lanes are combined using an OpenCV function using a weighting function. This produces the final image for the pipeline, which is shown here:

![alt text][image9]


## Potential Shortcomings

One potential shortcoming would be the inability to handle any non-straight roads. This pipeline assumes the lanes are straight, but curved or banked roads will break this pipeline since a straight line is fit to the data points during the Line Combination step.

Another short coming is the inability to handle outliers in the line fitting method, as using least-squares method is highly susceptible to outliers.

## Possible Improvements
To address the short coming of curved roads, higher order polynomials can be fit to the data points that will follow the curve of the road.

Secondly, the use of RANSAC rather than using least-squares will be beneficial to fit a single line to all of the edges.
