---
title: HydroFleet
layout: gridlay
excerpt: Human-centered AI and Robotics Lab
sitemap: true
permalink: /hydrofleet/
---

# HydroFleet
<iframe width="640" height="360" src="https://www.youtube.com/embed/-dp3m6c1-Rw?si=l2PLrPWbVW-1X-_Z" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## System Overview
HydroFleet is a multi-drone system that enables data-driven precision irrigation by generating high-resolution soil moisture mapping data, helping farms increase yields while reducing water and energy costs.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/hydrofleet-labeled.png" width="60%">

## Drone Platform
The drone platform is a multirotor UAV (HolyBro X500 V2) airframe equipped with two computing elements:

**1.** A flight controller (PixHawk 6X) responsible for low-level attitude stabilization, position hold, and waypoint navigation. The flight controller accepts high-level commands (e.g., go to waypoint, hold position, ascend, descent) and executes them using onboard inertial measurement, barometric, and GNSS sensors (DroneCAN H-RTK F9P Rover GPS).  

**2.** An onboard companion PC responsible for autonomous mission-level decision-making, including the sequencing of measurement sites and coordination of the soil measurement cycle with the soil sampling module. 

### Hardware
The figures below contain more information regarding the wiring and placement of various hardware used for this project.
<!-- show two images side by side -->
<div style="display: flex; justify-content: space-around;">
  <img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-1.png" width="45%">
  <img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-2.png" width="45%">
</div>

The drone platform also provides electrical power to the soil sampling module from its onboard battery, eliminating the need for a separate energy source on the module and minimizing suspended mass.

### Autonomy
Standard ground control software and flight controllers are already capable of tasks like basic waypoint navigation, but they lack the native logic to coordinate multi-stage mission sequences. Tasks that require tight synchronization between flight maneuvers – such as precision descent, CAN-based sampler actuation and real-time data logging – demand a higher-level autonomy layer. For this project, we implemented this higher-level orchestration using the PX4 ROS2 Interface Library. This library simplifies interacting with PX4 from ROS2 through the Control Interface, which allows for easy registration of custom modes written in ROS2. It implements the ModeExcutor() class, which acts as the orchestrator of multiple custom modes.

For HydroFleet, three custom modes are implemented:

**GoToWaypoint**: Navigates the drone to specified sampling coordinate.  

**SoilSample**: Descends the drone until the soil sampler is on the ground. Then, it actuates the sampler with a CAN signal and waits until measurement data are sent back.  

**CustomRTL**: Executes a custom landing behavior where the drone descends to land the sampler first. Then, the drone moves forward 1 meter and lands fully.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-3.png" width="50%">

To minimize the risk of damaging hardware, we used a custom Gazebo simulation environment for testing autonomous behaviors. Building upon the standard PX4 single-vehicle simulation, the environment was expanded to support multiple vehicles within a customizable farm environment. This digital twin allowed for rapid iterative testing, allowing us to identify and resolve any bugs in a controlled setting before deploying to the physical hardware.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-4.png" width="50%">

To streamline the multi-vehicle development workflow, we used a cloud-based workflow using balenaCloud. By grouping all drones into a single fleet, we can ensure that identical software environments are maintained across all units. Any updates in the code or environment are pushed to the entire fleet in one action. This significantly reduces configuration mismatches and ensures that every drone operates on the same validated software version during field tests.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-5.png" width="50%">

### Safety
When operating on the farm, the drones must be aware of its surroundings, since farmers could be working where it operates. Especially with an additional suspended load at the bottom, it is crucial that those in the drone's vicinity are aware of it to prevent injuries. To address this, we implemented three safety measures: one on the drone, another on the ground station, and the third as a handheld system control override.

#### Person Detection
The drone is equipped with a downward-facing camera that continuously monitors the area below it while running an existing deep learning model for person detection to check for humans. When a person is detected, the drone holds its position in place until the area below is clear.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-6.png" width="50%">

#### Audio-Visual Warnings
The ground station has a safety light tower that provides both visual and audio alerts to those in the vicinity. When all drones are on the ground, a solid green light is activated to indicate that it is safe to approach the area. Upon activation of the drones, a flashing red light accompanied with the buzzer is used to indicate that it is dangerous to be nearby. This is triggered whenever the drone is taking off or landing.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-7.png" width="50%">

#### Handheld System Control Override
The drone’s flight controller also provides various built-in safety features to handle unexpected issues that may arise during flight. When the drone reaches critical battery levels mid-flight, an automatic return-to-launch (RTL) is triggered. Similarly, RTL is triggered when connection with the base station is lost. This is particularly helpful in a farm environment where connection can be unstable. Lastly, the drone can be manually overridden by a human pilot at any point mid-flight in case of an emergency.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/drone-platform-8.jpg" width="30%">

## Soil Sampling Module
The soil sampling module is a novel component of HydroFleet and was designed from scratch throughout the project. It is a lightweight, self-contained electromechanical assembly that, once lowered to the ground surface, performs the complete measurement cycle autonomously under the direction of its onboard microcontroller. The module contains the following principal components, labeled in the figure below.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/soil-sampler-1.jpg" width="40%">

### Sampling Mechanism
The sampling mechanism consists of the auger and the soil moisture sensor. We used a capacitive sensor for soil measurements because it can accurately measure the soil moisture of the dirt while having the distinct cylindrical shape that is easy to work with. For the sensor to get deep into the ground, we selected an auger mechanism that could effectively penetrate the soil to the target measurement depth. To minimize payload weight, we discarded the original housing and retained only the metal probe, cut it short and TIG welded it back together. 

### Anchoring Mechanism
The anchoring mechanism utilizes a set of servos (MEUS Racing V2 Coreless RC Micro Servos--8.5KG) to deploy custom-designed anchors. These servos were selected for their high torque-to-weight ratio, providing the necessary force without adding significant mass to the assembly. To ensure efficient soil penetration, the anchor geometry features a tapered profile that begins with a narrow tip and widens progressively. The anchor's circular cross-section reduces drag during penetration and maximizes the surface area acting perpendicular to the auger's rotation. This also minimizes the amount the sampler lifts during anchoring. This design minimizes initial resistance while maximizing holding force once fully engaged. Each anchor is keyed directly to the servo horn for precise control and structural integrity. Additionally, the base of the system incorporates several spikes that improve stability and enhance initial ground contact upon landing, making it easier for the anchors to penetrate the soil.

### Actuation Mechanism
The actuation mechanism utilizes a continuous servo to drive a high-torque gear train. This primary gear interfaces with a secondary gear that rotates both the auger and the soil moisture sensor, enabling them to penetrate the ground effectively. To prevent wire tangling during rotation, the sensor is integrated with a slip ring on its top. The secondary gear subsequently drives a final gear containing an internal lead screw nut and as this gear rotates around a fixed lead screw, which translates the entire assembly linearly. Through iterative testing, a 1:1:2 gear ratio was established to synchronize the auger’s rotational speed with its descent rate. The platform remains stable during travel by sliding along two lateral support rods with nylon bushings for smooth movement. To maintain a low weight while ensuring durability, the assembly uses an aluminum shaft and 3D-printed gears, powered by a 25kg/cm servo capable of penetrating varying soil densities. 

### Communication
In order to drive the sampler servos and read values from the sensors, the sampler is controlled by an Arduino Nano mounted on the sampler. The entire sampling routine — driving the anchor servos, lowering the probe into the soil, reading moisture values, retracting the probe, and retracting the anchors — is executed from a script on the Arduino, simplifying the communication between the drone and the Arduino to a simple set of commands and the logging of data. By default, the sampler script is set to an `OFF` state where the servos are retracted. Upon receiving a specific command, the sampler either enters a `PAUSED` state (waiting for specific commands to drive individual servos, used for debugging and tuning) or the start of the autonomous sampling routine, which begins with `CLAW_EXTENDING`.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/soil-sampler-3.png" width="60%">

### Sensor Integration
To measure the moisture level of the soil we utilized the DSMM500 Precision Digital Soil Moisture Meter with Probe from General Tools. This is a capacitive sensor which measures dielectric constant of the surrounding medium. 
    
Hydrofleet requires a continuous data stream that could be processed and stored onboard our drone’s flight computer. To interface the barebone probe with our microcontroller, we designed a custom signal conditioning circuit using an LMC555 CMOS Timer configured in astable mode. This setup allows us to translate a physical change in soil capacitance into a measurable analog voltage reading that the drone can easily log.

From this dry baseline, we added specific masses of water to achieve our target Gravimetric Water Content (GWC) ratios of 0%, 15%, 20%, 25%, 30%, and 50%. For each ratio, the soil and water were mixed and packed to a uniform density, creating a homogeneous mixture. Both our custom astable circuit probe and an unmodified probe were inserted into the soil at each moisture level to record readings. By plotting the sensor's output voltage against the known GWC percentages, we generated a calibration curve shown below. 

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/soil-sampler-4.png" width="50%">

Our data show a strong linear relationship between voltage and moisture percentage, with $$R^2 = 0.9969$$. Notably, the unmodified factory probe exhibited the same linear trend across the targeted percentages, which successfully validated that our custom signal conditioning circuit was functioning correctly and accurately reflecting the soil's true dielectric shift.

## Multi-Drone Planner
In order to safely and efficiently fly a fleet with payloads over any farm field, robust path planning is necessary to ensure efficient labor and time distribution. HydroFleet integrates a path planning framework that takes into account the bounds of the drone’s flight endurance and the temporal and spatial constraints of any given farm field.  

Endurance is defined by a fixed energy capacity that undergoes constant depletion during operation, precluding mid-mission recharging. Each mission entails a takeoff from a home depot, completion of a set number of sampling points, and a return to the starting point before the battery depletes. The duty cycle is therefore limited by the energetic costs of both spatial transit and the hover duration required for soil sampling.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/multi-drone-planner-1.png" width="70%">

We break this problem into **four parts**.

**1.** Defining the number of cells to sample based on arguments like polygon (of the block of land) area, sampling time, flight speed, total mission time for each drone based on energy constraints, and number of agents  

**2.** Generating uniformly-spaced sampling points (Voronoi iterations)  

**3.** Generating optimal paths between the sampling points (Vehicle Routing Problem) based on the number of drones available  

**4.** Adding time delays to routes that overlap to ensure no collisions

### Defining Number Of Cells
The maximum possible time the drones can collectively spend sampling can be defined as the sum of the cumulative flight time for each drone divided by the sampling time. The actual flight time for each drone is calculated based on the area of the crop block, thus accounting for the time spent traveling between sampling points. The number of sampling points is incremented and returned when the actual flight time is less than the maximum possible time and divisible by the number of drones. Using this method, we get 12 total sampling points across a crop block in our partner farm.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/multi-drone-planner-2.png" width="40%">


### Sample Point Selection using Lloyd's Algorithm
Voronoi Iteration, also known as Lloyd’s algorithm, divides a plane into regions where each region contains exactly one seed point, and every location within that region is closer to its seed than to any other. With each iteration, the seeds spread apart and converge toward a uniform, equidistant distribution across the space. This approach is well-suited for soil moisture sampling for two reasons. First, the uniform distribution ensures that samples are spread evenly across the entire plot, giving the farmer a complete picture of soil conditions rather than data concentrated in one area. Second, because the points are approximately equidistant, the distance each drone needs to fly between stops is minimized, which makes the most of the available battery life.


### Multi-Drone Route Planning using Vehicle Routing Problem
The Vehicle Routing Problem (VRP) is an optimization method for finding the most efficient routes for a fleet of vehicles to visit a set of locations. In our case, it takes the Voronoi-generated sampling points and distributes them across the available drones, giving each drone an ordered sequence of waypoints that covers the plot within its battery limit. A generated trajectory may look like the following.

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/multi-drone-planner-3.png" width="40%">

These routes are exported as a json file that interfaces with the aforementioned ROS2 missions.

## Farmer Dashboard
The Farmer Dashboard serves as the primary interaction point between HydroFleet and the farmer. It was designed as a **mobile-friendly, intuitive user interface** that provides readings of measured values and plots trends across all current fields. This is presented in a user-friendly format that does not require the farmer to have a deep technical understanding of the process. These design decisions stemmed directly from conversation with farmers where they emphasized clear visuals and wanting to add custom sampling points. Our current main dashboard is shown in the image below. 

<img src="{{ site.url }}{{ site.baseurl }}/images/pjctpic/hydrofleet/farmer-dashboard-1.png" width="80%">

Within the main dashboard, the core features and information farmers need are displayed in a simplified format. We also developed the software to accommodate future additions, such as humidity, pH, and temperature measurements. The subpages then provide greater depth when needed, allowing the farmers to focus on a particular field, trends over time, or the raw data itself. The farmer dashboard runs as a React app using a Python FastAPI backend. Incoming drone and sensor data is posted to the API and stored in a PostgreSQL database via Supabase, and then received through the dashboard. The dashboard can be viewed through a mobile device, allowing for easy access outdoors.
