## Project: Search and Sample Return

**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

I defined two functions, `sample_thresh` and `obstacle_thresh` to identify rock samples and obstacles respectively. In `sample_thresh`, rock samples are defined as yellow objects, while in `obstacle_thresh`, obstacles are defined as darker objects (opposed to the ground/navigable terrain).

```python
def sample_thresh(img):
    color_select = np.zeros_like(img[:,:,0])
    above_thresh = (img[:,:,0] > 135) \
                & (img[:,:,0] < 205) \
                & (img[:,:,1] > 118) \
                & (img[:,:,1] < 178) \
                & (img[:,:,2] < 60)

    color_select[above_thresh] = 1
    return color_select

def obstacle_thresh(img):
    color_select = np.zeros_like(img[:,:,0])
    above_thresh = (img[:,:,0] < 160) \
                & (img[:,:,1] < 160) \
                & (img[:,:,2] < 160)

    color_select[above_thresh] = 1
    return color_select
```

![alt text][image1]

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

TODO

Describe in your writeup how you modified the process_image() to demonstrate your analysis and how you created a worldmap. Include your video output with your submission.

![alt text][image2]
### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

[image_grid]: ./calibration_images/example_grid1.jpg
[image_warped]: ./output/warped_example.jpg

`perception_step()` consists of the following steps:

**Define source and destination points for perspective transform and apply perspective transform**

We can convert a camera image, e.g., as the image below, to a partial 2D world map via *perspective transform*.

![Grid Image][image_grid]

This is the transformed image (a partial 2D world map!)

![Warped image][image_warped]

To achieve this goal, we will use this grid image as a calibration image. We locate the position of the four squares of the grid as source points and middle-bottom position in the 2D map as destination points. Then we apply `getPerspectiveTransform` and `warpPerspective` in OpenCV to transform image into 2D for us.

**Apply color threshold to identify navigable terrain/obstacles/rock samples**

The purpose of this step is to identify objects based on their color. In `color_thresh`, the navigable area is defined as objects that have lighter color. In addition, I defined `sample_thresh` and `obstacle_thresh` to identify rock samples and obstacles, as mentioned in **Notebook Analysis**.

**Convert map image pixel values to rover-centric coords**

To map a partial 2D map to the ground-truth map, we should convert it to rover-centric coordinates first. It allows the transformation

**Convert rover-centric pixel values to world coordinates**

This step consists of `rotate_pix` and `translate_pix`.

In `rotate_pix`, the yaw angle is first converted to radians. Then we can apply the rotation matrix with the yaw angle in radian.

In `translate_pix`, we get the absolute position in world coordinates first by dividing it by a scale, since a grid takes 10 pixel in our camera image, then we add the position vector to the position we get in the previous step.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

**Experimental Enviroment: Macbook Pro 13.3', OSX 10.12.4, Resolution: 800x600, Image Quality: Good, FPS: 25~32**

The original method will stop and take a turn when there is no enough navigable terrains. However, there exists some scenario where there is enough navigable terrains but the rover can't move, e.g., stucking by obstacles.

I enhance the by adding the mechanism of detecting stucking. The main idea is record an initial timestamp when forward mode in on, and if the speed is slow after being in forward mode for 5 seconds, it means getting stucked.

To solve the stucking problem, I add an new mode called `backward`. It can be triggered when a stucking problem is detected. The rover will go back until a speed limit is reached, and go back to the `stop` mode to either take a turn or moving forward

**Future work**

So far the picking up has not been implemented. The key point is how to make rover go directly to an object given its coordinate. Also the coverage of discovered map is not good enough. An intuitive approach may be giving the undiscovered area a higher weight when considering the steer angle (sort of like a DFS algorithm).
