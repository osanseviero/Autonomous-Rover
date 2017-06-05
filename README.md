[//]: # (Image References)
[image_0]: ./misc/rover_image.jpg
[main]: ./misc/example.png
[main2]: ./misc/example2.png
[main3]: ./misc/example3.png
[main4]: ./misc/example4.png
[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
![alt text][image_0]



This project is modeled after the [NASA sample return challenge](https://www.nasa.gov/directorates/spacetech/centennial_challenges/sample_return_robot/index.html) and is a first hand experience with the three essential elements of robotics, which are perception, decision making and actuation. The project is in a simulator environment built with the Unity game engine.  

## The Simulator
The first step is to download the simulator build that's appropriate for your operating system.  Here are the links for [Linux](https://s3-us-west-1.amazonaws.com/udacity-robotics/Rover+Unity+Sims/Linux_Roversim.zip), [Mac](	https://s3-us-west-1.amazonaws.com/udacity-robotics/Rover+Unity+Sims/Mac_Roversim.zip), or [Windows](https://s3-us-west-1.amazonaws.com/udacity-robotics/Rover+Unity+Sims/Windows_Roversim.zip).  

You can test out the simulator by opening it up and choosing "Training Mode".  Use the mouse or keyboard to navigate around the environment and see how it looks.

## Dependencies
You'll need Python 3 and Jupyter Notebooks installed to run this project.  The best way to get setup with these if you are not already is to use Anaconda following along with the [RoboND-Python-Starterkit](https://github.com/ryan-keenan/RoboND-Python-Starterkit). 


**Goals:**  
![alt text][main]
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

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
The main part here is to add the functions to detect other obstacles and the rocks. For the obstacles, an inverse of the ground color threshold was enough. A threshold of the range of (100, 100, 20) and (255, 255, 30) was applied. This is enough to detect when there is a rock. This was implemented in two new functions (obstacle_thresh and rock_thresh). While the original color_thresh for terrain was checking the pixel values to be above the thresh, the obstacle thresh checks that the values are smaller. Both 'and' and 'or' conditionals work for this function, but 'and' was chosen because it kept a high fidelity since it's sure that the values are of an obstacle. In order to detect which ranges to use matplotlib was used using some example images.

![alt text][main]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
The process_image() function is in charge, as the name implies, of receiving an image and processing it. First, it makes the percept transform. To this warped image, the three different filters are applid (terrain, obstacles, rocks). This data helps, after converting to world coords, to color the map. With the simulator, you can record a run in training mode, and then use the notebook to analyze each image recorded in the run and make a video with it. 

The steps were the next ones:
1. Define source and destination points for the perspective transform. 
2. Apply the perspective using the cv2 library functions getPerspectiveTransform and warpPerspective. This gives a view from the top, but you can change to a custom destination point to improve it.
3. Apply the three different color filters as explained in the previous section. This allows to differentiate between terrain, obstacles and samples to be gathered. This is then converted to rover coordinates (meaning that the rover position is the center bottom of the image):
4. This coordinates are then converted to world coordinates. This is accomplished by rotating, translating, and finally scaling the image. This is done for each of the three thresholded images
5. With this information, the world map can be updated and the mosaic image is populated.


![alt text][image2]
### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
The `perception.py` script has a perception_step() function that works pretty much in the same way than in the notebook (explained in the previous section). Instead of interacting with a Databucket class, the function interacts with a Rover class. To increase the fidelity, there's a margin of 2 in the roll and pitch (angles bellow 2 or over 358. This means that when the rover is rotating really fast, that data won't be mapped. This allowed to mantain a fidelity of almost 90%. An important thing to note is that the Rover angles and distances both for navigable terrain and rocks is updated at the bottom. This information is used by the decision script to know where to move.

The `decision.py` script works as the decision tree of the Rover. The first thing changed is that, when there are pixels activated with the rock filter, the rover will move in that direction. This causes some confusion when there are many rocks at the same time, but still works fine. To prevent the rover to go over the rock, it has a lower maximum velocity and the throttle is half. When it is close enough, it stops so it can pick the rock. Originally, this script only had a forward and a stop mode. I added a stuck mode. When the velocity is really low and there is still acceleration, it means that it's probably stuck with small obstacles. When this happens, the rover just steers a bit and tries to move again. Lastly, there is an open terrain in the right of the map which made the Rover move in circles since there was the same pixel density in the same direction. This was fixed by limiting the clip range in the forward mode. This decision tree allowed the Rover to map over 80% with high fidelity.


#### 2. Launching in autonomous mode your rover can navigate and map autonomously. 

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

The resolution used is 640x480, with fastest graphic quality. Most of the decision process was explained in the previous section. The things that worked well were limiting when the Rover was mapping depending of the roll and pitch values. The program mantains a fidelity of over 80% and maps over 80% of the map given enough time. It's also able to pick most, if not all, of the rocks. The FPS is around 30.

Some examples:

![alt text][main]
![alt text][main2]
![alt text][main3]
![alt text][main4]

