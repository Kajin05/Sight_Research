
# Sight Research: Directing Visually Impaired Users with Smart Glasses

This repository details my research process on directing the gaze of visually impaired individuals using smart glasses. This is a critical area of focus because if a visually impaired user is searching for an object that is outside the smart glasses' camera's field of view, the system cannot process the command effectively. 한국어
### Method

I propose a method to guide a user's gaze when they command AI-powered smart glasses to locate an object. The smart glasses would capture a wide-angle photo and process it to search for the requested object. If the object is not found, the glasses would direct the user to rotate in a specific direction. This rotation allows the glasses to capture another photo from a new viewing angle. This iterative process continues until the object is located, at which point the glasses would direct the user to reach in the general direction of the object. While not the sole method for object localization, I believe this approach offers the most efficient solution given the current technological limitations of smart glasses.

**[ADD GIF SHOWING HOW IT WOULD WORK]**

To assess the viability of this method, we must understand several key concepts:

- Camera Configuration
- Image Stitching
- Gyroscope Use

We will now examine each of these components and discuss their contribution to the process of object finding through AI glasses.

## Camera Angle of View

### One Camera

The camera's specifications, particularly its Angle of View (AOV), determine the range of the smart glasses' visual field. AOV refers to the maximum extent a camera can capture through its lens. A wider AOV provides a broader viewing angle but introduces more radial distortion, causing straight lines to appear curved. Conversely, a narrower AOV reduces distortion but captures less information. Below, I have set up a simulation to illustrate the outputs of different AOVs. The wide camera is set at a 120-degree AOV, while the narrow camera is set at a 60-degree AOV. 

<p align="center">
  <img src= photos/sight1.gif>
</p>

As demonstrated in the simulation above, the red ball appears in the wide-angle camera's view first, followed by the narrower camera. However, you can also observe that the wide-angle camera exaggerates depth, making the red ball appear farther away than its actual distance. Furthermore, wide-angle cameras are more susceptible to radial distortion; objects closer to the camera become increasingly stretched. The bottom simulation clearly illustrates this, showing that the square face is more distorted in the wide-angle camera's view compared to the narrow-angle camera's.

<p align="center">
  <img src= photos/sight2.gif>
</p>

<div align="center">

|      | Wide AOV Camera                        | Narrow AOV Camera                 |
| ---- | -------------------------------------- | --------------------------------- |
| Pros | - Can capture more information         | - Less likely to have distortion  |
| Cons | - Prone to radial and depth distortion | - Less information being captured |

</div>

The table above clearly demonstrates that both wide and narrow AOV cameras have distinct applications. For our purpose of quickly locating objects with an AI camera, a wide AOV camera would be more beneficial.

This choice is preferable because a wider AOV more closely approximates the natural field of human vision. This similarity in visual perception would allow the user to interact with the AI glasses more intuitively, as if the device were an extension of their own eyes.

---
### Two Cameras

Upon further inspection, I realized that if the smart glasses were to use two cameras, we could estimate the object's distance from the user through **stereo vision**. Knowing an object's depth is crucial for gaze processing; if an object is out of reach, the user needs to be aware that it's beyond their range.

Stereo vision operates by comparing two photos captured simultaneously by two cameras positioned at a fixed distance from each other. This technique leverages the similar triangle rule, where the line connecting two corresponding points in the separate images is parallel to the line between the two cameras.

<div align="center">
  
| ![sight3](photos/sight3.png) | ![sight4](photos/sight4.png) |
| ---------------------------- | ---------------------------- |

</div>

By comparing the real-world distance between the cameras to the digital distance of similar points within the images, we can apply the similar triangle rule, as shown in the equation below:

$$
 \frac{T}{Z}=\frac{T+{x}_l-{x}_r}{Z-f} \to Z=\frac{f\cdot T}{{x}_l-{x}_r}
$$

Using this equation, I simulated and tested how different AOV glasses could determine depth. In the simulation below, I evaluated the ability of 60 AOV and 120 AOV cameras to ascertain the depth of a red ball. The measured depth is displayed in the bottom-left corner, and the views from the left and right cameras are shown. I calculated the percent error by comparing the measured values at the 0.5-meter and 1-meter marks. Note that each grid square represents 0.5 meters.

**60 AOV Stereo Camera**
<p align="center">  
  <img src= photos/sight5.gif>
</p>

**120 AOV Stereo Camera**
<p align="center">  
  <img src= photos/sight6.gif>
</p>

<div align="center">

| 60 AOV      |                  |                    |
| ----------- | ---------------- | ------------------ |
| *Distance*  | *Measured Value* | *Percetange Error* |
| 0.5 m       | 0.528 m          | 5.6%               |
| 1.0 m       | 1.026 m          | 2.6%               |
|             |                  |                    |
| **120 AOV** |                  |                    |
| *Distance*  | *Measured Value* | *Percetange Error* |
| 0.5 m       | 0.911 m          | 82.2%              |
| 1.0 m       | 2.501 m          | 150.1%             |

</div>


As the table above illustrates, the percentage error in depth increases drastically from a 60 AOV to a 120 AOV, indicating that a wider angle of view makes accurate depth perception more challenging. However, there are limitations to this conclusion. The depth calculation requires focal length, yet the vision sensor in the simulation provides only AOV and camera resolution. This necessitated calibrating the FOV using known distances and then calculating the focal point. Since the focal length might not be precisely accurate, it introduces uncertainty into these results.

**One Wide, One Narrow?**
I considered testing a combination of one wide-angle and one narrow-angle camera, with the idea that the wide-angle camera could spot objects while the narrow-angle camera calculated depth. Although technically feasible with prior calibration, this approach would not be practical. It would be more efficient to utilize two wide-angle cameras and calibrate them beforehand for accurate depth estimation. This setup would offer a broader field of view while still providing precise depth information.
## Image Stitching

If we use two cameras to measure depth, we can also put their two images together to create a single panorama using a technique called image stitching. This approach would simulate human vision more accurately and give the AI a better understanding of an object's location relative to where the user is looking. A continuous panoramic photo offers a superior representation compared to two separate images.

OpenCV, an open-source computer vision software library, provides image stitching functionalities. The full stitching tutorial is avaliable in this [github](https://github.com/OpenStitching/stitching_tutorial/blob/master/docs/Stitching%20Tutorial.md), however here is a simplified overview of the process involves several steps:
Lets say we are given these four images and asked to create one photo. Let's say they are images one through four from left to right.
<p align="center">  
  <img src= photos/sight8.gif>
</p>

1. Resizing: Images are initially resized to a medium resolution. This makes processing these images easier.

2. Feature Detection: The system then identifies unique features or elements within these images that may also be present in other images.
<p align="center">  
  <img src= photos/sight7.gif>
</p>

3. Feature Matching: These features are matched across corresponding images to determine which images have a high confidence of overlap. These matches are based on confidence. For example, image 1 and 2 has high confidence that they are a match. However, image 4 would have low confidence with all other images, showing that image 4 is not part of this panorama.
<p align="center">  
  <img src= photos/sight9.gif>
</p>

4. Subset Creation and Warping: Subsets of relevant images are created, and these photos are then warped to ensure correct composition.

5. Cropping and Stitching: Finally, the images are cropped and seamlessly stitched together, prioritizing areas with minimal interference.
<p align="center">  
  <img src= photos/sight10.gif>
</p>
<p align="center">  
  <img src= photos/sight11.gif>
</p>

The common practice of security companies using two cameras to achieve a 180-degree field of view (AOV) demonstrates the viability of image stitching for creating seamless panoramic images. 
## Gyroscope Use
Gyroscope is needed for the AI glasses to know where the user is looking at. When the user is turning their head to look for an object, a new image should be taken when the user is facing a different angle where they were originally facing. A gyroscope measure the change in rotational axis, allowing to take the photo when the timing is just right. 
**IMAGE**

With just measuring rotational change, 3 degrees of freedom (3 DOF) would suffice, but when we want to track the AI glasses, we would need a 6 DOF gyroscope/accelerometer tracker. Addtionally, because these trackers need to intergrate to get the position values, there are a lot of errors when calculating for the real position. Hence, to get the most accurate positional value, GPS is needed to avoid inaccuracy. 

## Method
