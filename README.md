# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[firstpass]: ./examples/solidWhiteFirstPass.jpg "First Pass"
[secondpass]: ./examples/solidWhiteCurveExtrapolated.jpg "Extrapolated"
[challenge1]: ./examples/Challenge_FirstPass.jpg "Challenge First Pass"
[challenge2]: ./examples/Challenge_FirstPass_NotExtrapolated.jpg "Challenge First Pass Minus Extrapolate"
[testImage1]: ./examples/solidWhiteMod.jpg "Modified test image"
[testImage2]: ./examples/solidWhiteMod_lanes.jpg "Modified test image Lanes"
[challenge3]: ./examples/Challenge_Improved.jpg "Challenge improved"
[challenge4]: ./examples/Challenge_ExtraLines.jpg "Challenge Extra Lines"

---

### Reflection

### 1. Initial Pipeline

My pipeline consisted of essentially what was contained in the lesson.  It followed these 5 steps:
1. Grayscale the image
2. Gaussian Blur
3. Canny Edge Detection
4. Region Mask
5. Hough Transform

All of this was fairly simple to achieve using the provided helper functions and suggested threshold values from the course material.

The initial image as a result of this looked like this:
![alt text][firstpass]

After this, I updated the draw_lines function with an "extrapolate" option.  This first split the lines into left lines and right lines using the slope, took the average of those groups into a single line, and then extrapolated that line.  To extrapolate the line, I took the y endpoints from the region of interest, and then solved for the x values using the original slope formula (x1 = x2 - (y2 -y1)/slope).  The result was this:
![alt text][secondpass]

This allowed me to draw lane lines successfully on all input images and most videos.  However, this solution started to show some cracks when applied to the challenge video.  The first thing I noticed was that my region of interest was based on hardcoded pixel values, and the challenge video was a different resolution than the other images and videos  (1280x720 vs 960x650).  I found this by updating my draw_lines() function to have an option to draw the region of interest.  This option also allowed me to tune the region of interest a bit more to better encapsulate the lanes in all images and videos.  However, it was quickly apparent that my extrapolated lane lines were not correct:
![alt text][challenge1]

I first turned off extrapolation, just to see what lines were being detected.  It turned out, lines were being found everywhere, from the median divider, to shadows, the front of the hood, etc:
![alt text][challenge2]

I suspected some of this could have been cleaned up by tuning the pipeline thresholds/variables, but instead I took the approach of logically separating the lines based on context.  To assist with this, I edited the initial test image to add some "fake" lane lines that I wanted to ignore:
![alt text][testImage1]

First, I removed all the horizontal lines by filtering any line with a slope to close to 0 (i.e. less than .4 or greater than -.4).  This was a huge portion of the noise in the challenge video.  To get rid of the remaining parallel noise lines, I searched the web for the best ways to remove outliers, and came across the [Median Absolute Deviation](https://en.wikipedia.org/wiki/Median_absolute_deviation) (i.e. "MAD") algorithm.  There were many freely available python snippets of this algorithm which I put together into my own function.   Using these methods, I was able to prune the fake lines from my test image easily, and adding back extrapolation lead to the real lane lines.  
![alt text][testImage2]

When re-applying this to the challenge video, the result was much improved: 
![alt text][challenge3]

However, I still noticed the lines jumping in some spots.  I turned the extrapolation off again, and found some trouble spots where my outlier detection was failing:
![alt text][challenge4]

It seemed there were some situations where more non-lane lines were detected than the lane lines themselves.  This lead to the lane lines being the outliner.  This seemed like a reasonable stopping point for my current approach.


### 2. Potential shortcomings of my pipeline

As discussed above, my approach for the most accurate lane lines was to simply filter non-lane lines based on the context.  This was mostly effective, however, parallel edges detected by the pipeline within the region of interest could in some cases be more prevalent than the lane itself.  Also, without any history, the line is re-drawn on each frame, so temporary changes in the edge detection have a major impact on this approach.  


### 3. Possible improvements to my pipeline

To improve upon the above lane detection pipeline, I could have taken a few approaches:
* _**Reduce noise by tuning the region of interest specifically for the car:**_  For example, in the challenge video, the camera seemed mounted further back than the others, and slightly off to the right.  In a real car, the region of interest could be tuned more accurately to avoid reading edges in other lanes and/or the shoulder.  It could also be slightly forward to avoid noise from the hood of the car (refections, hood edge, etc), and with less distance to avoid seeing edges in other lanes when the road curves.  I did not tune this in my pipeline so that it could remain applicable to all input images/videos.
* _**Tune edge detection for specific colors:**_ To avoid detecting shadows, road imperfections, etc, the pipeline could have been tuned to ignore edges on colors other than yellow or white.
* _**Use historical data:**_ This is obviously outside the scope of this exercise, but I suspect the next step that a real autonomous car engineer would take is to use the previously detected lanes as a predictor for the next frame.  Real road lanes don't suddenly shift.  They follow a relatively smooth path.  Thus, the newly detected line could be drawn with the same slope as the previous frame, and adjustments could be made based on newly detected edges with some pre-defined weight.  Major outliers could be rejected and-or the entire result could be thrown out until the next frame.  Again, since lanes follow a smooth path, even at highway speeds a car could afford to drop a few frames of lane detection and just stay on its original path.
