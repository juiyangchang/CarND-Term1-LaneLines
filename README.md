# **Finding Lane Lines on the Road** 
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./figures/masking.png "Masking"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. 

1. Convert the image to gray scale
2. Gaussian filtering with kernel size of 5
3. Perform Canny Edge detection with `low_threshold = 50` and `high_threshold = 100`
4. Masking    
      Masking is applied to find only edges within particular region of interest. The masking size I used scales according to the image dimensions.  Below I show how large the mask would be like in an image with blue dashed lines.
    
![][image1]

5. Hough Transform 

  The Hough Transform parameters are listed in the following: 

  `rho = 1`, `theta = np.pi/180` (that is, 1 degree), `threshold = 40`, `min_line_length = 20` pixels, `max_line_gap = 150` pixels

   I set `min_line_length` and `max_line_gap` to a relatively large number as we want to find actual lines (thus `min_line_length` = 20 pixels) but also being able to connect some line sections that don't have enough of pixels (thus `max_line_gap` = 150 pixels).  These two parameters along with `threshold` are probably worth tuning so that the pipeline would work with any image dimensions.  But I did not move into that direction.

6. Drawing lines (the `draw_lines()` function)
  
  The line drawing approach is divided into three stages: filtering, grouping and averaging, and extrapolation
  
      1. Filtering: In filtering and grouping, I first remove lines with slopes between -0.5 and 
          0.5 (slope is the ratio of height (y) to width (x)).  The idea behind this is that it is unlikely for 
          an actual lane line close to the car to have absolute slope smaller than 0.5. After all, the car 
          is moving forward and lines with slopes really deviating from the cars' body edges is not going to
          be helpful.
          
      2. Grouping: Next, the lines are partitioned into groups by their slopes.  Lines with slopes from 
          0 to 0.5 (excluding 0.5) is assigned to a group, lines with slopes from 0.5 to 1 (excluding 1) is
          assigned to a second group and so on.  The idea behind this is that even a line in the same image can
          have multiple detected edges from Canny edge detection and Hough Transform. Chances that are multiple
          lines with slightly differing slopes actually belong to a same lane line. Once the groups are formed, 
          I do another voting to take the top two group with most lines.  The idea behind this is that I think
          in the masked area, the two most populated slope groups are highly likely to be the two different lane 
          lines we are interested of.
          
      3. Extrapolation: We then form group average slope and average bias for each of the groups. The lines are 
          then drawn from the bottom of the image to 7/11 of image height, the corresponding x axis can be
          computed with the slopes and biases.

### 2. Identify potential shortcomings with your current pipeline

My current pipeline is really unstable.  I did trial and error with the mask dimension and the Hough transform parameters to make it sort of work for the challenge video. Even then the lines still fluctuate a lot. Slight tweak of these parameters would make it complete breakdown.

The grouping method I used in my `draw_lines()` is actually not really robust.  Changing other parameters may result in the top two lines being on the same side of the car.

Another shortcoming of the approach is that, when
there is shadow or somehow a lane line is partly
unclear, the approach is also really sensitive to the parameters.  


### 3. Suggest possible improvements to your pipeline

I think some preprocessing techniques of the images might help but I am still looking for it.  I tried
histogram equalization over the RGB channels prior to
gray image conversion but it will make the performance even worse.  

I think a procedure that will definitely be helpful
is to use Kalman filtering or similar techniques to
utilize locations or slopes of the lane lines in the previous video frames. If the pipeline is really certain about the location of the lane lines in some of the previous video frames, filtering will pass the information to the current frame and we can fuse all information together to decide the location of the lane lines in the current video frame.