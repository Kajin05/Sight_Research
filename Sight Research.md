# Sight Research: Directing Visually Impaired Users with Smart Glasses

This repository details my research process on directing the gaze of visually impaired individuals using smart glasses. This is a critical area of focus because if a visually impaired user is searching for an object that is outside the smart glasses' camera's field of view, the system cannot process the command effectively.
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

![sight1](photos/sight1.gif)

As demonstrated in the simulation above, the red ball appears in the wide-angle camera's view first, followed by the narrower camera. However, you can also observe that the wide-angle camera exaggerates depth, making the red ball appear farther away than its actual distance. Furthermore, wide-angle cameras are more susceptible to radial distortion; objects closer to the camera become increasingly stretched. The bottom simulation clearly illustrates this, showing that the square face is more distorted in the wide-angle camera's view compared to the narrow-angle camera's.

![sight2](photos/sight2.gif)

|      | Wide AOV Camera                        | Narrow AOV Camera                 |
| ---- | -------------------------------------- | --------------------------------- |
| Pros | - Can capture more information         | - Less likely to have distortion  |
| Cons | - Prone to radial and depth distortion | - Less information being captured |

The table above clearly demonstrates that both wide and narrow AOV cameras have distinct applications. For our purpose of quickly locating objects with an AI camera, a wide AOV camera would be more beneficial.

This choice is preferable because a wider AOV more closely approximates the natural field of human vision. This similarity in visual perception would allow the user to interact with the AI glasses more intuitively, as if the device were an extension of their own eyes.

---
### Two Cameras

Upon further inspection, I realized that if the smart glasses were to use two cameras, we could estimate the object's distance from the user through **stereo vision**. Knowing an object's depth is crucial for gaze processing; if an object is out of reach, the user needs to be aware that it's beyond their range.

Stereo vision operates by comparing two photos captured simultaneously by two cameras positioned at a fixed distance from each other. This technique leverages the similar triangle rule, where the line connecting two corresponding points in the separate images is parallel to the line between the two cameras.

| ![sight3](photos/sight3.png) | ![sight4](photos/sight4.png) |
| ---------------------------- | ---------------------------- |
By comparing the real-world distance between the cameras to the digital distance of similar points within the images, we can apply the similar triangle rule, as shown in the equation below:
$$
 \frac{T}{Z}=\frac{T+{x}_l-{x}_r}{Z-f} \to Z=\frac{f\cdot T}{{x}_l-{x}_r}
$$
Using this equation, I simulated and tested how different AOV glasses could determine depth. In the simulation below, I evaluated the ability of 60 AOV and 120 AOV cameras to ascertain the depth of a red ball. The measured depth is displayed in the bottom-left corner, and the views from the left and right cameras are shown. I calculated the percent error by comparing the measured values at the 0.5-meter and 1-meter marks. Note that each grid square represents 0.5 meters.

**60 AOV Stereo Camera**
![sight5](photos/sight5.gif)

**120 AOV Stereo Camera**
![sight6](photos/sight6.gif)

| 60 AOV         |                      |                        |     | 120 AOV              |                        |
| -------------- | -------------------- | ---------------------- | --- | -------------------- | ---------------------- |
| *Distance (m)* | *Measured Value (m)* | *Percetange Error (%)* |     | *Measured Value (m)* | *Percetange Error (%)* |
| 0.2            | 0.1916               | 4.20                   |     | 0.3453               | 72.7                   |
| 0.3            | 0.2956               | 1.47                   |     | 0.5008               | 66.9                   |
| 0.4            | 0.3875               | 0.625                  |     | 0.6677               | 66.9                   |
| 0.5            | 0.5283               | 5.66                   |     | 0.9112               | 82.2                   |
| 0.6            | 0.5813               | 3.12                   |     | 1.1129               | 85.5                   |
| 0.7            | 0.9178               | 31.1                   |     | 1.2523               | 78.9                   |
| 0.8            | 0.7926               | 0.925                  |     | 1.4309               | 78.7                   |
| 0.9            | 0.8719               | 3.12                   |     | 1.6694               | 85.5                   |
| 1.0            | 1.026                | 2.50                   |     | 2.501                | 150                    |

As the table above illustrates, the percentage error in depth increases drastically from a 60 AOV to a 120 AOV, indicating that a wider angle of view makes accurate depth perception more challenging. However, there are limitations to this conclusion. The depth calculation requires focal length, yet the vision sensor in the simulation provides only AOV and camera resolution. This necessitated calibrating the FOV using known distances and then calculating the focal point. Since the focal length might not be precisely accurate, it introduces uncertainty into these results.

**One Wide, One Narrow?**
I considered testing a combination of one wide-angle and one narrow-angle camera, with the idea that the wide-angle camera could spot objects while the narrow-angle camera calculated depth. Although technically feasible with prior calibration, this approach would not be practical. It would be more efficient to utilize two wide-angle cameras and calibrate them beforehand for accurate depth estimation. 

As seen with the percentage error of the 120 AOV camera setup, the values are in a relatively compact range, from 66.9% to 85.5% (if we consider 150% an outlier). This consistent error indicates that we can either calibrate the depth calculation beforehand or undistort the images. This setup would offer a broader field of view while still providing precise depth information.
## Image Stitching

If we use two cameras to measure depth, their two images can also be combined into a single panorama using image stitching. This approach would more accurately simulate human vision and provide the AI with a better understanding of an object's location relative to the user's gaze. A continuous panoramic photo offers a superior representation compared to two separate images. Furthermore, a larger field of view would reduce the need for the user to turn their head frequently because of the increased likelihood of the object being within the camera's view.

OpenCV, an open-source computer vision software library, provides image stitching functionalities. The full stitching tutorial is avaliable in this [github](https://github.com/OpenStitching/stitching_tutorial/blob/master/docs/Stitching%20Tutorial.md), however here is a simplified overview of the process involves several steps:
Lets say we are given these four images and asked to create one photo. Let's say they are images one through four from left to right.
![sight8](photos/sight8.png)

1. Resizing: Images are initially resized to a medium resolution. This makes processing these images easier.

2. Feature Detection: The system then identifies unique features or elements within these images that may also be present in other images.
![sight7](photos/sight7.png)

3. Feature Matching: These features are matched across corresponding images to determine which images have a high confidence of overlap. These matches are based on confidence. For example, image 1 and 2 has high confidence that they are a match. However, image 4 would have low confidence with all other images, showing that image 4 is not part of this panorama.
![sight9](photos/sight9.png)

4. Subset Creation and Warping: Subsets of relevant images are created, and these photos are then warped to ensure correct composition.

5. Cropping and Stitching: Finally, the images are cropped and seamlessly stitched together, prioritizing areas with minimal interference.
![sight10](photos/sight10.png)
![sight11](photos/sight11.png)

The common practice of security companies using two cameras to achieve a 180-degree field of view (AOV) demonstrates the viability of image stitching for creating seamless panoramic images. 
## Gyroscope and Speaker Use

**Gyroscope**
A gyroscope is essential for the AI glasses to determine the user's gaze direction. When the user turns their head to locate an object, a new image needs to be captured once they're facing a different angle than before. A gyroscope measures changes in rotational axis, enabling the system to take a photo at precisely the right moment.

![sight12](photos/sight12.gif)

This is a simulated example of how the glasses would direct the user to find an object, in this case, a red sphere. Since we know the combined AOV of the two cameras and their initial orientation, we can determine when the user changes direction. When the user moves outside the AOV of the initial picture, a new image is captured to continue searching for the object. This process repeats until the object is found. Once we have an image of the object, we can calculate its depth using stereo vision. Additionally, we can determine the object's offset from the center of the image by finding the average pixel position of the object and measuring its distance from the central axis.

![sight13](photos/sight13.png)

**Speakers**
Building on the location data, we can guide users to an object using spatial audio cues delivered through stereo speakers. This involves employing the Head-Related Transfer Function (HRTF), which models how our ears perceive sound from specific points in space.

![sight14](photos/sight14.png)

For instance, if an object is located to the user's right and slightly below chest level, we can use HRTF to generate a "ping" sound that appears to originate from that precise direction. This spatialized audio enables the user to intuitively reach towards the object.

HRTF works by calculating how an incoming sound wave, from its source, is filtered by the diffraction and reflection caused by the user's head before it reaches their ears. By creating digital filters that mimic these real-world acoustic effects, we can accurately recreate these spatial cues, providing an immersive and directional auditory experience.

## Putting it All Together

```mermaid
graph TD
	A(왼쪽 카메라) -->|왼쪽 이미지| C[이미지 처리] 
	B(오른쪽 카메라) -->|오른쪽 이미지|C 
	C --> |파노라마/이미지| D{메인 제어} 
	E(자이로스코프) --> |방향| D 
	D --> |객체 위치|F[스피커] 
	F-->|방향 안내|G(사용자) 
	G --> |입력|D
```

This understanding confirms the initial method's feasibility. By using two wide Angle of View (AOV) cameras, we can achieve a panoramic view with depth sensing, effectively simulating human vision. The user would then look around to find an object. When the glasses detect a new viewing direction via the gyroscope, a new photo is taken. Once the object is located, we can determine its position and guide the user with an audio ping indicating its estimated location. This system would allow users to find objects solely with AI glasses.