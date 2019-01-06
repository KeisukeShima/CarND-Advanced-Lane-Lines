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

[image1]: ./output_images/undistort/straight_lines1.jpg "Undistorted"
[image2]: ./output_images/undistort/straight_lines1.jpg "Road Transformed"
[image3]: ./output_images/binary/straight_lines1.jpg "Binary Example"
[image4]: ./output_images/birds_eye/straight_lines1.jpg "Warp Example"
[image5]: ./output_images/detect_line/straight_lines1.jpg "Fit Visual"
[image6]: ./output_images/result/straight_lines1.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

You're reading it!

### Project file

[Project Here](./workspace/main.ipynb)

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./workspace/main.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at `pipeline()` in `./workspace/main.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `birds_eye_view()`, which appears in cell 7 in the file `./output/main.ipynb`.  The `birds_eye_view()` function takes as inputs an image (`binary_img`), and define source (`src`) and destination (`dst`) points in the function.  I chose the hardcode the source and destination points in the following manner:

```python
    src = np.float32([
        [691, 450],
        [1070, 700],
        [230, 700],
        [595, 450],
    ])

    dst = np.float32([    
        [1010, -100],
        [1010, 720],
        [354, 720],
        [354, -100]
    ])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 691, 450      | 1010, -100    | 
| 230, 700      | 354, 720      |
| 1070, 700     | 1010, 720     |
| 595, 450      | 354, -100     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used a function called `find_lane_pixels()` in the functipn `fit_polynomial()` in my code in `main.ipynb`.  
I modified hyperparameters such as number of windows, wondows margin and minimum number of pixels in the find_lane_pixels(). I used sliding window approach with those hyperparameters.  
It worked correctly with all sample images.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in functions `measure_curvature_real()` and `vehicle_offset()` in my code in `main.ipynb`  

* Curvature

I defined conversions in x and  from pixels space to meters. Vertical conversion is set to 30 meters per 700 pixels because the lane length is about 30 meters and lane length in the image is about 700 pixels.  
Horizontal convension is set to 3.7 meters per 700 pixels because lane width is 3.7 meters and lane width in the image is about 700 pixels.

* Offset

I defined the center is half of the width of the image. I calculated the center of the lane from the position of the left lane and the position of the right lane and decided the offset from the difference from the center of the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in my code in `main.ipynb` in the function `birds_eye2original()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)  
Here's a [link to my video result (challenge)](./challenge_video_output.mp4)  
Here's a [link to my video result (harder_challenge)](./harder_challenge_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?  (このプロジェクトの実装で直面した問題や問題について簡単に話し合います。あなたのパイプラインはどこで失敗する可能性がありますか？より堅牢にするために何ができますか。)

[English]  

* This time I decided while adjusting the coefficients of the perspective transform manually, so I have to adjust again when the camera changes. Originally you should automate this.
* In [this movie](./project_video_output.mp4), the median strip is erroneously recognized as the left lane in the middle.
* As in [this movie](./harder_challenge_video_output.mp4), the lane will appear outside the area of the perspective transform on a winding road, so pipeline processing does not work well.

[Japanese]  

* 今回はperspective transformの係数を手動で調整しながら決めたので、カメラが変わったら再度調整しないといけない。本来はこれを自動化するべき。
* [このムービー](./project_video_output.mp4)では、中盤で中央分離帯を左レーンと誤認識している。
* [このムービー](./harder_challenge_video_output.mp4)のように、曲がりくねった道ではperspective transformの領域の外にレーンが出てしまうため、パイプライン処理がうまく働かない
