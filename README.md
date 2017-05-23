## Advanced Lane Finding
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

[image1]: ./images/undistort_output.png "Undistorted"
[image2]: ./images/test1.png "Original Image"
[image7]: ./images/test2.png "Road Transformed"
[image3]: ./images/binary_combo_example.png "Binary Example"
[image4]: ./images/warped_straight_lines.png "Warp Example"
[image5]: ./images/color_fit_lines.png "Fit Visual"
[image6]: ./images/example_output.png "Output"
[video1]: ./project_video.mp4 "Video"

---


### 1. Camera Calibration

The code for this step is contained in the second code cell of the IPython notebook located in "AdvancedLaneFinding.ipynb" .  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion corrected image 

The function `cv2.calibrateCamera()` returns the camera matrix, distortion coefficients , row and translation vectors. I used the camera matrix, distortion coefficients retrieved from the Camera calibrate function and passed the parameters to `cv2.undistort` function to retrieve the distortion corrected image below 

![alt text][image7]

#### 2. Color transforms, gradient to create the thresholded image 
I tried to convert the image from RGB to HLS format. I found that `S` component helped to identify the lanes clearly compared to `H` and `L` format. Futher, I applied cv2.sobel() on the `S` component to get the both colored and combined binary. I used a combination of color and gradient thresholds to generate a binary image (thresholding function can be found in cell 14 in `AdvancedLaneFinding.ipynb`.  Here's an example of my output for this step.  

![alt text][image3]

#### 3. Perspective transformed image 

The code for my perspective transform includes a function called `perspective_transform`, in the ipython notebook `AdbvancedLaneFinding.ipynb`.  The `perspective_transform()` function takes as inputs an image (`img`). This image will be given to `cv2.warpPerspective()` function to get the warped image. I chose source and destination points below to get the Perspective Transform Matrix. 


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 240,686   | 300,700      | 
| 1060,686      | 1000,700     |
| 738,480     | 1000,300     |
| 545,480      | 300,300      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Identifying Lane pixels 

On the warped binary image, the peaks of the histogram was identified to extract the midpoint, left base and right base positions. The sliding window method was applied to construct the rectangle windows on the warped image. Then `np.polyfit()` function was used to fit the second degree polynomial. The function to identify the lane pixels is fit_poly and can be found in `AdvancedLaneFinding` notebook. 
The sample image with the lane pixels identified is as shown below, 
 
![alt text][image5]

#### 5. Radius of curvature and position  

The code for identifying the radius of curvature can be found in the ipythion notebook in the function 'fit_poly()'.  The lane is assumed tobe 30 meters ling and 3.7 meters wide. 
The lane positions, for left and right, will be used to fit new polynomials to x,y in world space. The radii of the curvature will be calculating the by using the formula,

   <pre>
    Rcurve = (1 + (2Ay + B)^2)^1.5 / |2A| 
   </pre> 
​
​​For finding the distance from lane center, the camera was assumed to placed at the center of the car such that the lane center is the midpoint at the bottom of the image between the left and right lanes identified from the previous step.  
​​ 

#### 6. Example image of the result

The image below is the output from the fit_poly() function 

![alt text][image6]

---

### Pipeline (video)

#### 1. Video Link 

Here's a [link to my video result](video_output.mp4)

---

### Discussion

I had some problems in processing the video frames. The pipeline would not perform well if the video frames is not identified corrently.  To make the implementation more robust, I would include the below steps, 
1. Try convolution approach to identify the lanes 
2. Include sanity checks to identify the best polynomial fit 

