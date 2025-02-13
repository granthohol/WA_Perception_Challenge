# Wisconsin Autonomous Perception Challenge Spring 2025
## Grant Hohol

## Overview
This program is designed to detect the boundaries of a straight path defined by orange cones in an image. The input is a .png image containing small orange cones, and the output is the same image with lines drawn to represent the estimated left and right boundaries of the path. The program uses OpenCV for image processing, NumPy for numerical operations, and Matplotlib for some image output in a Jupyer Notebook.

The methodology involves several steps, including color thresholding, contour detection, clustering, and line fitting. Below is a detailed explanation of each step.

## answer.png
<img src="answer.png" width="600" height="800">

## Libraries Used
- cv2
- numpy
- matplotlib.pyplot

## Methodology
### Step 1: Image Loading and Preprocessing
1. Load the image
2. Convert to HSV
- Image is converted from default BGR to HSV becasue HSV is more suitable for color-based segmentation

### Step 2: Color Thresholding
1. Create mask
- A range of HSV values is defined for the orange cones and is passed to cv2.inRange() to create a mask that highlights the orange cones. Determining the bounds proved to be a little difficult and you can read more about that in the following section.

### Step 3: Contour Detection
1. Find Contours
- Contours are detected from the mask using cv2.findContours()
2. Filter Contours
- The mask contained some highlighted pixels that were not cones, so the findContours() function was picking those up and detecting them. So, I decided to filter the contours based on a minimum contour area. After a little trial and error, I landed on a minimum area of 100 which successfully filtered out the noisy contours and isolated and kept the cone contours. 

### Step 4: Centroid Calculation
1. Calculate and store centroids for each contour
- For each contour, the centroid is calculated using image moments (cv2.moments()). The centroid represents the approximate location of each cone in (x,y) coordinates. Each of these centroids are stored in a list.

### Step 5: Clustering (K-Means)
1. K-Means Clustering
- The centroids are clustered into two groups using K-means clustering (cv2.kmeans()). We can set 2 as our number of clusters parameter because we want two lines and know the cones are in two left and right groups.
2. Assign Clusters
- The cv2.kmeans() function cannot immediately tell us which group is the left cones and which is the right, so we assign the groups to "left" or "right" based on their average x-coordinates. The cluster with the smaller average is labeled the left cluster and the other as the right cluster. I then output these clusters to ensure that each left centroid has a lesser value than each right centroid, and they do. 

### Step 6: Line Fitting and Drawing 
1. Fit Lines to Clusters
- For each cluster, a line is fitted through the centroids in each cluster using linear regression (cv2.fitLine()).
2. Calculate Line Endpoints
- In order to draw a line over the original image, we need the endpoints of the line. So, two end points are calculated for each line based off the return value of cv2.fitLine() which gives us the slope and a point on the line. Each line spans the height of the image and starts from the bottom of the image. 
3. Draw Lines
- Using the endpoints for the two lines, the left and right boundary lines are drawn on the original images copy using cv2.line().


## What I Tried and Why It Did Not Work
1. I had a fair amount of trial and error with setting the bounds for Hue, Saturation, and Value for the cv2.inRange() function to create the mask. It took me awhile to realize that what worked best was actually to just set hue to 0. This was a little unintuitive at first, but I think it works because if you look at the HSV image, there is nothing as bright/light as the cones regardless of color. I would like to learn some techniques to go about this process in an easier and less tedious way. 
2. Even after realizing this to create the mask, there were still some problems with creating the contours from the mask because it was picking up some contours that were not cones (what I believe is the exit sign in the original image). Trying to filter this out with the bounds proved impossible which makes sense because the exit sign and the cones are basically the same color, so I had to take a different approach. My new approach was to filter out contours that weren't above a certain area (noisy), and this proved successful. 