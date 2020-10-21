# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report



---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

To check the code of the whole pipeline you need to check the process_image method. I didn't modify the draw_lines function

My pipeline consisted of 3 iterations and each iteration consisted of 4 steps.
The iteration spes are: 
(1) get prepared image (get_prepared_image method)
(2) get lines (get_lines method)
(3) get border lines (get_border method)
(4) transform border lines into the solid borders

In the first step, I just convert an image to grayscale, apply gaussian blur and canny transform, and then cut off the region of interest. 

In the second step, I transform an image from the first step into the array of lines by Hough transform and then separate these lines into the left and right group relative to the center of the image.  

The third step applies to the left and right array of dots separately. This is the biggest and the most important step and, for its part, it is separated into 4 substeps. The first one is to filter out lines that have a too low or big slope. I try to do it with strong conditions, if there are no lines that can meet our requirements, we try to use weak conditions, and if no result again, we just use unfiltered input. 
Then we look for only pairs of parallel lines(with some tolerance), which I averaging and filtering again with the code from the first step.
On the third substep, I need to convert lines into the array of dots.
And at the finish, I need to choose 2 dots with the longest distance between them.

And the last but not least step is to transform these borders into a solid line from the bottom of the screen to the end of traffic lines.  
On this step, I obviously transform lines into a polynomial, and then I generate new lines using these polynomials, which lie from the bottom of the screen to the most top y coordinate of these two lines.

Actually, these 4 steps work not bad for two first videos, but I faced several problems connected with the challenge video.
The first problem is that sometimes, the video is too low contrast so my thresholds are too strong to detect even solid lines. But on the other hand, applying low thresholds to the whole video leads us to the dramatic quality of lines decreasing.  
To solve this I try to detect lines with 3 groups of settings. First time I run an iteration of our pipeline with the strongest thresholds. Next,  I run it with the same settings but with a high contrasted image. And the last try is to run the original image with the low thresholds. After each iteration, I check, do both lines have a good slope. If both are well enough the process is finished, else I run the next iteration and trying to take the best result of both.

The second problem is the camera position in the challenge video. I decided to solve this problem by just cutting off the bot of the video, which contains only the hood of the car.

But even after all these steps, the lines in the video are not well enough smoothed, so to solve this, the last issue I decided to accumulate previous results and for the current image use the result of the next expression:
line = k\*accumulated line + (1-k)\*line 
That approach is close to the Kalman filter, but I used just hardcoded k coefficient, instead of using the Kalman algorithm.



### 2. Identify potential shortcomings with your current pipeline

The main shortcoming is the performance in hard cases. Because I analyze every single image from one up to 3 times it may be a problem in real-time processing. 
Also, because of accumulating results, images with errors accumulate too, so a sequence of errors may ruin even good enough borders.

### 3. Suggest possible improvements to your pipeline

Several possible improvents: 
- try to automaticaly detect the hood
- try to cut off the middle of the region of interest
- avoid using try catche as flow control in the main pipeline