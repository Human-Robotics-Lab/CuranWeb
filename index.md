---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
permalink: /
---

Curan is a medical toolkit written in C++ that simplifies integration with third-party dependencies and provides solutions for your custom medical needs. The project is developed under the umbrella of [Human Robotics Lab](https://github.com/Human-Robotics-Lab) organization

You can find the source code for Curan on GitHub:
[Curan](https://github.com/Joaopmoliveira/CuranSDK)

Now, let's continue our adventure into programming. This is hopefully a detailed view of how to use and understand the project layout. 

If you want to get started, read the [build instructions]({{ site.baseurl }}/get_started/) on how to compile Curan on your personal machine. Currently, we have only tested Curan on Windows and Ubuntu, although most libraries should work out of the box on macOS.

If you want to see some of the applications we’ve developed with this collection of libraries, please check out some of our work in [the demos folder]({{ site.baseurl }}/demos/).

If you already have Curan installed on your machine, take a look at some of the [tutorials]({{ site.baseurl }}/tutorials/) we provide to guide you through the Curan API. Here are some videos of what the sofware allows you to build:

# Trajectory Planning

Medical teams utilize a broad range of planning algorithms that automatically fuse multiple image modalities, and simplify visualization of targeted anatomies.
Curan ships its own path planner, where we utilize volume rendering techniques to facilitate and streamline the planning task for end-users.

<div>
<video muted autoplay  width="700" controls>
    <source src="{{ site.baseurl }}/assets/videos/VolumePlanning.mov" type="video/mp4">
    <p>Your browser does not support the video element.</p>
</video>
</div>

# Spatial calibration with untracked phantoms

For our navigation platform its important that we are able to systematically calibrate the ultrasound-based system without expensive optical trackers. Through
a distinct approach from the literature we are able to estimate a minimal representation of the wire poses and the pose of the ultrasound scanner, rigidly attached to the flange 
of the robot.

<div>
<video muted autoplay  width="700" controls>
    <source src="{{ site.baseurl }}/assets/videos/spatial_calibration.mov" type="video/mp4">
    <p>Your browser does not support the video element.</p>
</video>
</div>

# Volume reconstruction in real-time

To find the anatomical coordinate frame with respect to the robot we must directly tackle the registration problem. Although we have multiple strategies to solve this problem
our prefered method is based on volumetric registration using mutual information. To do this we must take each bi-dimensional US scan, recreate a 3D volume where we can evaluate
the overlap metric.

To guarantee that the code can be executed in realtime we use a multithreaded implementation 

[!Render Time - US image size]
render times as a function of width of US scan and number of threads
![side by side]({{ site.baseurl }}/assets/images/render_times.png)

To guarantee that the medical team can interpret the quality of the reconstructed volume we render its whilst the cranium is reconstructed

<div>
<video muted autoplay  width="700" controls>
    <source src="{{ site.baseurl }}/assets/videos/volume_reconstruction_wires.mov" type="video/mp4">
    <p>Your browser does not support the video element.</p>
</video>
</div>

