## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./output_images/output_5_1.png "Road Transformed"
[image3]: ./output_images/output_5_2.png "SobelX Binary"
[image4]: ./output_images/output_5_3.png "S channel Binary"
[image5]: ./output_images/output_5_4.png "S channel Color Binary"
[image6]: ./output_images/output_5_5.png "Combined Binary"
[image7]: ./output_images/frameno_365persp.jpg "Persp Transform"
[image8]: ./output_images/straight_lines1.jpg "Img-8"
[image9]: ./output_images/output_15_1.png "Img-9"
[video1]: ./project_video_processed_111.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the caliberate_camera() function inside camera class given in the IPython notebook.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here is the distortion corrected image which is generated using the undistort_image() function inside Camera class given in the IPython notebook:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I have transformed the image to HLS color space inside threshold_image() function in Camera class. I chose S channel since it is less invariant to lighting fluctuations. Futher I converted the RGB image to grayscale followed by histogram equalization which helps in detecting white lines better. I thresholded the SobelX gradient operator on the histogram equalized image and combined it with the thresholded version of the S channel image to generate the final binary image.  Here's an example of my output for this step.

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is included in perspective_transform() function inside camera class in IPython notebook. This function takes as inputs an image (`img`) and perspective transform matrix (`M`) which is generated using the source (`src`) and destination (`dst`) points.  I chose to hardcode the source and destination points in the following manner:

```python
self.source_cord = np.float32([[608, 445], [669, 441], [1091, 714], [226, 711]])
self.dest_cordinates = np.float32([[200,0], [1080,0], [1080,719], [200,719]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 608, 445      | 200, 0        | 
| 669, 441      | 1080, 0       |
| 1091, 714     | 1080, 719     |
| 226, 711      | 200, 719      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Lane pixels and polynomial fit is done in locate_lines() function in Lanes class in IPython notebook. The code is mostly borrowed from Udacity classroom implementation except for few changes to make it more robuts. The polynomial fit coefficients are averaged over last five frames so that it gives averaging effect in the video and the detected lanes don't jump around too much in the video. Further, a complete full scan is triggered whenever there is no fit points detected in the right or left lanes. Further, a complete full scan is also triggered whenever right detected points cross-over into left detected points and vice-versa. I akcnowledge that the current implementation is not the best and avenues for improvements are given in the last section.

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

ROC is calculated using the function computeROC() in Lanes class in IPython notebook. It is based on the method as explained in the class. Position of the vechicle with respect to center is calculated as the distance between the image center and the lane center transformed into real world units.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step using plot_lanes() function in Lanes class in IPython notebook. Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_processed_111.mp4)

Here's a [link to challenge video result](./project_video_processed_222.mp4)

Here's a [link to harder challenge video result](./project_video_processed_333.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The basic approach used is what has been told in classroom except for few tweaks to make it more robust. Averaging the fit coefficients for the last few frames are beneficial, triggering a full scan whenever sanity checks fails. There are several scope of improvements listed below:

1. Full scan trigger is time consuming, hence need to be triggered only when deemed necessary. I'm currently triggering it whenever there are no pixels detected in left or right lanes or whenever right lanes cross into left and vice-versa. This check can be made bit more relaxed based on failure over few successive frames since frames are captured at 30fps

2. Left and right lanes should be seperated by atleast minimum number of pixels apart. We can perhaps give more confidence to the lane where more points are detected with smaller spread and use this criteria to filter off the lanes getting detected in-between real lanes due to gradient

3. Averaging length for fit coefficients should be based on ROC for the previous frame, this would help in updating the fit faster if ROC is changing faster

4. The entire project can be solved using neural network if we could feed in supervised training data in the form of line fit coefficients along with the image from the center camera. Need to try it!
