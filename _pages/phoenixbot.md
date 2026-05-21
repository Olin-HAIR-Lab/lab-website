---
title: Phoenixbot
layout: gridlay
excerpt: Human-centered AI and Robotics Lab
sitemap: true
permalink: /phoenixbot/
---

{% include head-custom.html %}

# Phoenixbot

<iframe width="640" height="360" src="https://www.youtube.com/embed/JOZpxB6JnlA?si=yLsPdz-Yy21n72vF" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## System Overview
Phoenixbot is an autonomous robot for precision weeding. The system combines color-based segmentation with clustering and deep learning-based classification for weed detection and localization, a delta-arm weeder for targeted weed removal, and a vision-based crop row-following algorithm for field navigation.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/phoenixbot/phoenixbot-labeled.png" width="70%">

## Mechanical Weeder

<iframe width="640" height="360" src="https://www.youtube.com/embed/vYnQtPHCzmE?si=XD0NERGulzXpWBhE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/phoenixbot/weeder-arm-labeled.png" width="80%">

### Weeder Arm: Hardware

![]({{ site.url }}{{ site.baseurl }}/images/pjctpic/phoenixbot/mech-weeder-1.png){: style="width: 400px; float: left; margin: 5px 20px 5px 0px;"}
The mechanical weeder removes weeds identified by the plant identification system. To ensure both precision and efficiency, a delta arm was chosen due to its high-speed and precise operation. The delta manipulator comprises of a fixed base plate and a mobile end-effector platform connected by three link arms. Each arm consists of a bicep and a forearm, with biceps connected to the base via revolute joints. Three stepper motors spaced at 120° drive the biceps. The forearms are connected to the biceps and the end effector platform via spherical joints, ensuring parallel alignment with the base.

<div style="clear: both;"></div>

### Weeder Arm: Software
![]({{ site.url }}{{ site.baseurl }}/images/pjctpic/phoenixbot/mech-weeder-2.png){: style="width: 400px; float: right; margin: 5px 0px 5px 20px;"}
The weeder arm can operate in two modes: autonomous and manual. In autonomous mode, the system receives the cartesian coordiantes of weeds, plans an appropriate path, and commands the arm to move accordingly. The arm then uses the claw to remove the weed before returning to home position. In manual mode, the system allows for joystick control, enabling the user to direct the arm's movements. Motor commands are sent to an Arduino, which executes them to control the motors. This system ensures efficient and precise weeding in both autonomous and manual operations.

<div style="clear: both;"></div>

### Weeder Claw
![]({{ site.url }}{{ site.baseurl }}/images/pjctpic/phoenixbot/mech-weeder-3.png){: style="width: 400px; float: left; margin: 5px 20px 5px 0px;"}
The weeder claw, serving as the end effector of the weeder arm, combines PETG plastic components with sheet metal to form a robust structure. Actuated by a single servo motor, it efficiently picks weeds. Various designs for the tongs were tested to determine the optimal configuration for the final version.

<div style="clear: both;"></div>

### Weeder Claw: Test Kit
![]({{ site.url }}{{ site.baseurl }}/images/pjctpic/phoenixbot/mech-weeder-4.png){: style="width: 400px; float: right; margin: 5px 0px 5px 20px;"}
To allow for parallel testing of the weeder claw and weeder arm, we developed an independent claw test kit that mimicked the robot's interface with the claw. It allowed single-hand vertical operation, allowing rapid testing of various tong and claw designs. Initially, a simple system with a button, continuous servo, and Arduino Nano was used to open and close the claw, but it didn't accurately simulate how it would function on the weeder arm. We added a current sensor to measure the servo's load, enabling detection of when the claw was fully opened or closed based on current draw.

<div style="clear: both;"></div>

### Evaluation
![]({{ site.url }}{{ site.baseurl }}/images/pjctpic/phoenixbot/mech-weeder-5.png){: style="width: 400px; float: left; margin: 5px 20px 5px 0px;"}
The weeder arm must be able to accurately move the weeder claw into positions given by the plant ID pipeline to ensure it can remove weeds without damaging crops. Using a testing grid with known locations, the weeder arm was commanded to move the claw to specific positions in Cartesian space. After each motion, the final position was compared to the true position to assess accuracy. During twenty-five trials, the weeder arm was capable of positioning the claw with a mean error of 6.009 mm from the commanded position.

<div style="clear: both;"></div>

## Plant Identification

### Segmentation
To distinguish crops from weeds, we implemented a plant segmentation pipeline using a color-based approach combined with DBSCAN clustering. Initially, HSV thresholding isolates green pixels in the image. The image is then converted to binary, with green pixels as white and everything else as black. To reduce noise, morphological operations are applied: a 3x3 opening operation and a 10x10 closing operation. Finally, DBSCAN groups the white areas into clusters, with each cluster representing a different plant.

### Classification
Once different plants are identified from the segmentation pipeline, each plant is cropped from the image and processed through a pre-trained neural network from Pl@ntNet (Garcin et al., 2021). We selected the ResNet18 model, which has 3000 different plant species it can identify. To minimize false negatives, a list of expected crops is defined. If the model's top five outputs with the highest confidence include a crop from this list, the plant is identified as that crop. Any plant not labeled as a crop is then classified as a weed.

### Localization
Once all weeds are identified, their coordinates are sent to the weeder arm for removal. Pixel coordinates of the weed centers, calculated during segmentation, are converted to real-world coordinates using Intel's Depth Camera D435 module with its built-in depth sensor. This achieves ≤ 11 mm accuracy in both x and y directions.

### Evaluation
To validate the segmentation pipeline, precision, recall, and mean IOU values of the predicted bounding boxes were calculated and compared to the ground truth. The segmentation pipeline showed an average precision of 0.69, recall of 0.70, and IoU of 0.68. For the classification pipeline, a confusion matrix for lettuce and weeds was calaulated. The result showed an accuracy of 0.88, precision of 0.91, and recall of 0.85.

## Autonomous Navigation

### Crop Row Following
The robot uses an Intel Realsense camera and OpenCV to maintain a straight path down a crop row. It isolates plants using HSV thresholding to mask non-green pixels, then crops the image to reduce error and computation time. Crop row lines are determined by analyzing green pixel ratios and smoothing peaks to identify rows. Assuming three crop rows, the system tries to minimize squared distances to the lines and calculates the vanishing point, which sets the robot's heading angle. A PID controller converts this angle into angular velocity, adjusting the linear velocity proportionally to ensure precise navigation.

### Object Detection
Since farmers will be working around the crop rows where the robot operates, the robot needs to be aware of its surroundings. We use Hokuyo's URG-04LX laser sensor, which allows coverage for the front and most of the sides. When the scanner detects an object, it enters one of three states: Danger, Warning, or Safe, depending on the proximity of the object. Based on the state, the robot will stop, slow down, or continue moving, respectively.

### Lighthouse
To visually indicate the robot's "safety state," the object detector sends its state to the LED lighthouse, which then activates the corresponding colored light. Additionally, a buzzer is triggered when the robot is in either the Danger or Warning state, ensuring that nearby individuals can hear it even if they cannot see the lights.

### Evaluation
Using a 30 second video running at 30 frames per second and running each frame through our line detector at 320x240 resolution we got all three crop lines in 904/924 frames or 97.8% of the frames. We can also look at a cloud of calculated vanishing points to see how the estimated heading point changes over the 900 frames. We can see that with the exception of 11 outlier points almost 99% of our points match what seems to be the center of the crop row. Remember here that only the x value of the point matters as it is what is used to calculate the angle.
