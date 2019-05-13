# **Finding Lane Lines on the Road**
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

## Overview
When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes act as our constant reference for where to steer the vehicle. Naturally, one of the first things we would like to do in developing a self-driving car is to detect lane lines using an algorithm automatically. In this project, I take a very first step to discover lane lines in images using Python and OpenCV.

## Objectives
The goals of this project is to make a pipeline that finds lane lines on the road.

[image1]: ./solidWhiteRight.jpg "Road"

[image2]: ./image_output/solidYellowCurve2.jpg "YellowCurve"

[image3]: ./image_output/solidYellowLeft.jpg "YellowLeft"

![alt text][image1]

---
### Reflection
### 1. Pipeline
My pipeline consisted of 5 steps. First, I converted the images to grayscale; then I used Gaussian blur technique with a kernel size of 5 to make the image little noisy. After that, I used the Canny method to detect edges passing low and high threshold values. The edge image was then processed to mask ROI with the required color channel. The resultant image was then again transformed using Hough transformation to obtain a line image. This image is then combined with the original color image to get the last lane marked image. 
Please see the snippet of pipeline code:

* gray = cv2.cvtColor(result,cv2.COLOR_RGB2GRAY) # processing the masked image
* blur_gray = gaussian_blur(gray,kernel_size)
* edges = canny(blur_gray, low_threshold, high_threshold)
* masked_edges=region_of_interest(edges,vertices)
* line_image=hough_lines(masked_edges, rho, theta, threshold, min_line_length, max_line_gap)
* lines_edges=weighted_img(line_image, image, α=0.8, β=1., γ=0.)

To draw a single line on the left and right lanes, I modified the draw_lines() function by to add a separate thick left and right lanes. Please see the below steps: 

* Segerate the lines based on their slope to Left Lane(Negative slope) or Right Lane.
* 2. Calculate a point on the line this is center of the line.
* 3. for left lane, use the point slop function (y-y')=m(x-x') to find top left and bottom left (x1 and x2) points using y1 and y2 as 60% of height and 100% of the height 
* 4. Repete this action to find x1, and x2 for right lane. 
* 5. In case of lines containing "NaN" values use the last good known x1 and x2 points.

Please see the snippet of code:

    for line in lines:
        for x1,y1,x2,y2 in line:
            #cv2.line(img, (x1, y1), (x2, y2), color, thickness)
            if x2==x1:
                continue # ignore a vertical line
            if y1==y2:
                continue #ignore a horizontal line
            slope = (y2-y1)/(x2-x1)
            center=[(x2+x1)/2,(y2+y1)/2] 
            if slope < 0: # y is reversed in image
                lm.append(slope)
                lc.append(center)
            else:
                rm.append(slope)
                rc.append(center)
    #average over all right/left center,slope values to get single center and slope
    r_slope=np.sum(rm)/len(rm)
    l_slope=np.sum(lm)/len(lm)
    r_center=np.divide(np.sum(rc,axis=0),len(rc))
    l_center=np.divide(np.sum(lc,axis=0),len(lc))
    
    #leftlane
    if np.isnan(l_center).any():
            x1=previous_frame_lines['l_x1']
            x2=previous_frame_lines['l_x2']
            cv2.line(img, (x1, y1), (x2, y2), color, thickness)
    else:
        previous_frame_lines.pop('l_x1', None)
        previous_frame_lines.pop('l_x2', None)
        y1=int(img.shape[0])
        x1=int(l_center[0] -(l_center[1]/l_slope) + (y1/l_slope))
        y2=int(img.shape[0]*.6)
        x2=int(l_center[0] -(l_center[1]/l_slope) + (y2/l_slope))
        #print(x1,y1,x2,y2)
        cv2.line(img, (x1, y1), (x2, y2), color, thickness)
        previous_frame_lines['l_x1']=x1
        previous_frame_lines['l_x2']=x2
    
    #rightlane    
    if np.isnan(r_center).any():
        x1=previous_frame_lines['r_x1']
        x2=previous_frame_lines['r_x2']
        cv2.line(img, (x1, y1), (x2, y2), color, thickness)
    else:
        previous_frame_lines.pop('r_x1', None)
        previous_frame_lines.pop('r_x2', None)
        y1=int(img.shape[0])
        x1=int(r_center[0] -(r_center[1]/r_slope) + (y1/r_slope))
        y2=int(img.shape[0]*.6)
        x2=int(r_center[0] -(r_center[1]/r_slope) + (y2/r_slope))
        #print(x1,y1,x2,y2)
        cv2.line(img, (x1, y1), (x2, y2), color, thickness)
        previous_frame_lines['r_x1']=x1
        previous_frame_lines['r_x2']=x2

![alt text][image2]
![alt text][image3] 

### 2. Identify potential shortcomings with your current pipeline
There are many potential shortcomings of this project are  
* a) What would happen when we pass thru a construction zone;
* b) Unclear lane markings;
* c) Overlapped lane markings;
* d) Bad weather etc.   

### 3. Suggest possible improvements to your pipeline
A possible improvement would be to use the different light spectrum to see objects and markings better. Instead of finding lanes we should define an ROI, a rectangular space which varies based on the speed of the car, weather condition, and location, etc.

### 4. Final Video
[Find Lane Lines](https://www.youtube.com/watch?v=yHErCYu-f0A)
