# **Finding Lane Lines on the Road**
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_images/blur.jpg "Bilateral Filtering"
[image2]: ./writeup_images/gray.jpg "Grayscale"
[image3]: ./writeup_images/edges.jpg "Canny Edge Detection"
[image4]: ./writeup_images/roi.jpg "Region of Interest Truncation"
[image5]: ./writeup_images/hough_lines1.jpg "Hough Lines"
[image6]: ./writeup_images/hough_lines.jpg "Weighted Mean and Slope Filtering"
[image7]: ./writeup_images/solidYellowLeft.jpg "Extrapolate and Render"

---


### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consists of the following 7 steps:

**Step 1:  Bilateral Filtering** <br />
I chose to perform bilater filtering instead of gaussian blur. This gave marginal improvement in performance due to the edge preserving nature of bilateral filter. I was also able to use much higher kernel than I would have used with gaussian blur without removing some edges. This gave slightly better performance in the challenge video.
Used function bilateral_filter a wrapper on top of open cv bilateralFilter function.

![alt text][image1]

**Step 2:  Convert to Grayscale**<br />
We convert the image to grayscale to identify all edges without regard to color boundaries. I used function grayscale to convert from RGB space to gray scale.

![alt text][image2]

**Step 3: Canny Edge Detector** <br />
Canny edge detector runs through the image and extracts all points where contrast is high
![alt text][image3]

**Step 4:  Region of Interest Truncation** <br />
We truncate region in image on which to run hough transform. Since, edges beyond the lane markers are not needed to draw lane markers. This technique saves CPU compute time of running expensive hough transform on this.

![alt text][image4]

**Step 5: Hough lines extraction** <br />
Hough transform is able to pick all the group of points which lie on straight lines. In the hough domain, this would be a point where lots of sin waves intersect. We select this point by dividing the hough domain into cells and then counting number of sine waves intersecting in a grid cell. Our threshold of minimum number of points picks only such grids where there are atleast so many sine waves. This way we pick points which form longer lines. 

![alt text][image5]

**Step 6:  Weighted Mean, Hough line slope filtering** <br />
We iterate over all the slopes of each of the line segments returned by hough tranform. 

To filter out the important line segments and to prevent noise in our mean computation, we seperate the image into 2 parts; left half and right half. We are only interested in slopes that are negative in the left half and positive slopes in the right half. Since mean is very sentive to outliers, this gives us better mean line slopes. Additionally, we compute weighted mean proportional to the length of the lines. This way shorter lines have lower probability of influencing the mean slope and prevent small edges from changing mean too much. Otherwise, small irregularities on the road tend skew our lane lines. 

![alt text][image6]

**Step 7: Extrapolate and Render over original image** <br />
We then use draw line function to extrapolate our mean slope to the roi boundaries using simple line formula. This gives us our final output.

![alt text][image7]


#### Challenge
1. Make the ROI relative to different Video resolution
2. Added bilateral filter to smooth out shadows but preserve lane edges
3. Added slope filters to filter out slopes that are horizontal or vertical 


### 2. Identify potential shortcomings with your current pipeline


**1.** Concrete and yellow line detection 
Proposed Pipeline has some issues rendering correctly over the concrete portion(where yellow and road contrast are reduced) challenge video 

**2.** Sensitivity to occlusion
Proposed Pipeline is very sensitive to occlusion and other objects in ROI. If there is another car in the ROI then our mean slope might be wrong. 

**3.** Smoothness of rendering in Video
Lane lines follow the lane markers but don't follow smoothly

**4.** Curve detection 
Proposed Pipeline does not handle curves well. Mountainous roads will be a challenge


### 3. Suggest possible improvements to your pipeline

**1.** Maintain a running average of previous frames so the lane lines are rendered more smoothly, also this allows to maintain lane lines when short occlusions or road changes occur like the challenge video.

**2.** To identify curved lane markers we need to see if hough transform can be modified to identify points lying on arcs of ellipses


