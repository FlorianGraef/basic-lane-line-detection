#**Finding Lane Lines on the Road** 

##Florian Graef

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images/generated/10/solidWhiteCurve.jpg_final_10.png "Phase 3"
[image2]: ./test_images/generated/05/solidWhiteCurve.jpg_final_05.png "Phase 1"
[image3]: ./generated_images/canny.png "Grayscale"
[image4]: ./generated_images/masked.png "Grayscale"
[image5]: ./generated_images/hough.png "Grayscale"
[image6]: ./generated_images/final.png "Grayscale"
<img src="./generated_images/final.png" alt="Drawing" style="width: 400px;"/>

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

![alt text][image2]

I started developing my pipeline to identify lane lines by first chaining up the offered helper functions in the order that we learned in the classes. My intention was to first establish a basic pipeline that works to refine it later. So chained up the following steps:

1. Conversion of the RGB image to grayscale
2. Gaussian blurring with kernel size of 7
3. Canny edge detection (lower threshold=50, higher threshold=150)
4. Masking the image to only consider a certain area of interest where we expect lane lines
5. Hough transform (rho = 1, theta = np.pi/180, threshold = 8, min line len = 50, max line gap = 30) 

I then worked on optimizing the hough transform parameters with the idea to already get long continuos lines bridging the gaps of the lane lines. That worked well on the single test images but when applying this to the video resulted in longer periods of time where no lane lines at all were detected because the now strict criteria of the hough transform were not easy to satisfy at all times.
Consequently the parameters were changed to 30, 5,  3 so that generally more lines would be present for further processing.
Since many small lane lines are suboptimal for determination of the position of the car with regard to the lane lines the next steps included to reduce the number of lanes found to two, one left and one right.
The right left distinction was done using the slopes of the found hough lines. Upon right left categorization based on slope the many left and many right lines were merged into one each by calculating and averaging their intercepts with the horizontal lines at Y=0.6*imshape[0] and Y=imshape[0] of the image. The following image shows the resulting averaged and exrapolated lane lines.
The combination of more, shorter hough lines being merged into a single left and right lane worked well for the videos one and two but in the optional challenge video my pipeline was struggling to detect lanes. Especially the shade on the road caused troubles but as well the lane detection seemed unstable and the lanes were jumping around.

From the Q&A video and the hint of the possibly helpful OpenCV functions, inculding cv2.inRange, the idea to just look at white and yellow colour as input for the pipeline was taken.
This has proven to help a lot with continuosly detecting lane lines, especially in the shade, but still the lane lines were a bit unstable and jittery.
Conclusively the pipeline got amended and with preprocessing consisting of gaussian blurring and filtering for yellow and white colour before the pipeline outlined above was used on the image data.

###2. Identify potential shortcomings with your current pipeline

The most obvious pitfall of this pipeline is that it can only detect straight lane lines. That works reasonably well on highways as roads have to be relatively uncurvy to permit high speeds. On more curvy roads or in cities this lane line detector would quickly cause problems. For example at intersections it would not be able to "see" the intersecting lanes. How could a car turn into intersecting roads? This would not be possible with this detector.

Another challenge are roads where the lane lines are not clearly identifiable. This can happen when the road colour is close to the lane line colour. This could happen in dusty, sandy areas or when road and lane lines ar completely coverd by snow. In the challenge video there is a part of the road where the tarmac is much ligher coloured. This immediately cause the detected lane lines to become very jittery.

Even though I have not come across any lane lines which were neither yellow nor white, that does not mean they do not exist. With the colour ranges applied the pipeline becomes less flexible and will likely struggle on roads with unusual lane markings. 

Another problem would be roads without any lane lines at all. I think the most flexible approach, would be to detect road space (smooth solid, surface) measure it and identify lanes by dividing up the available road area. Where lane lines are availbe the previous considerations are unnecessary and the lines can be detected and used.

Furthermore other vehicles could obstruct lane markings. Even though a self driving car would keep distance and hence would see lane markings on a highway there may be drivers cutting in with too little distance. And this would be a more common scenario in (slower) urban traffic.
In these cases temporal information of lane lines would help and as well as generally support to avoid jittering detected lines. This could be implemented saving the information of lane lines in the last frames.

###3. Possible improvements

Assuming straight lines it seems that all lane lines meet at focal point on the horizon. That could be exploited to filter out all predicted lane lines which do not point to this focal point. Furthermore lane lines can be expected to be approximately equidistant to each other providing enoug space for a car to travel wihin two lines. This could be used to guess lane lines where detecting lines does not work well.

Further tuning of the colour selection ranges could be beneficial to address the issue of noise in video three and maybe it is possible to filter out hough lines which do not agree with the other hough lines. Outlier detection could be explored together with k-means clustering. 

I suspect that the colour filtering causes its own issues as some parts of a pale, dusty road might be identified as yellow, which would result in a strong colour change (black to yellow/white) detected by the canny edge detector.

The pipeline would be more flexible by operating on the contrast edges alone and would have a fair chance to detect lane lines in more irregular colours. The general purpose of lane markings requires a good contrast. Exploiting just this feature by using canny edge detection should ensure generalizability of the model which may be required in special conditions.

Instead of simply averaging the intersections of extrapolated hough lines linear regression combined with outlier detection could be applied to the two hough line groups to filter out noise.
