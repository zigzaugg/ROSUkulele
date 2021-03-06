# rosukulele package
#### Northwestern University
#### ME 495: Embedded Systems in Robotics
#### Chenyang Meng, Levi Todes, Yuwei Xia, and Joshua Zaugg

## Introduction

The goal of this package is to have a RethinkRobotics Sawyer robot autonomously tune a ukulele. The ukulele, a custom pick, and a custom tuning peg are all placed in pre-designated locations on a platform along with an alvar tag. Using the built-in head camera, Sawyer senses the alvar tag and knows the relative locations of each of the aforemetnioned tools, it tunes the ukulele. The tuning process can be broken down into the following pattern:

1. Pick up the pick
2. Pluck the designated ukulele string
3. Listen to the produced pitch and find error from expected pitch
4. Set down the pick
5. Pick up the tuning block
6. Move to the designated tuning peg on the ukulele
7. Turn the peg by an ammount proportional to the pitch error
8. Set down the tuning block
9. Repeat steps 1-8 until pitch error is below a specified tolerance
10. Repeat steps 1-9 until on each of the 4 ukulele strings

After completing all of these steps, the expectation is to have a tuned ukulele, fit for use by the great man himself, Israel Kamakawiwoʻole. This entire process can be initiated by running the command `roslaunch rosukulele main.launch`

## Nodes

[tune.py](https://github.com/zigzaugg/rosukulele/blob/master/src/tune.py) - This node is used as the main control node of the process. It defines the relative (to the ar tag) locations of all the tools in robot's task space. It further defines the service calls, to the services initiated in the other nodes, in such a way so as to achieve the tuning of the Ukulele. It controls what stage of the tuning process to complete and in what order to do so.

[fetch.py](https://github.com/zigzaugg/rosukulele/blob/master/src/fetch.py) - This script starts the 'fetch' node and makes the 'move_to' service available. This node is built off of the `intera_examples` script `go_to_cartesian_pose.py`. Because of this, the service definition `MoveTo` accepts a call in the form of a string and parses arguments in the way the script it's built on used to parse command line arguments. 

[getPitch.py](https://github.com/zigzaugg/rosukulele/blob/master/src/getPitch.py) - This `pitches` node subscribes to the `/pitch` topic published by the `tuner`. It also establishes the service `get_pitch`. When the service is requested, it waits for 3 messages, then takes the pitch average of the next 3 messages and returns them. 

[gripper.py](https://github.com/zigzaugg/rosukulele/blob/master/src/gripper.py) - This `gripper` node is built off of the `intera_examples` script called `gripper_keyboard.py`. It creates the service `grip_pls` which can be called with either `o` to open the grippers or `c` to close the grippers. 

[transform.py](https://github.com/zigzaugg/rosukulele/blob/master/src/transform.py) - This `uku_tf_listener` node creates the `transformation_matrix` service. The service accepts a string that corresponds to x y z coordinates in the frame created by the alvar tag (often reffered to as the `/Ukulele` frame but really called `/ar_marker_2`) and uses `tf.LookupTransform` to convert these coordinates into the base frame. It returns these coordinate in the `/base` frame.

[tuner.py](https://github.com/zigzaugg/rosukulele/blob/master/src/tuner.py) - This `tuner` node is adapted from [python-tuner](https://github.com/mzucker/python-tuner) by Mark Zucker. It uses Fourier transforms on every 0.1s of audio to get the dominant pitch and publishes this on the `/pitch` topic. 
[tune.py](https://github.com/zigzaugg/rosukulele/blob/master/src/tune.py)

## External Packages

[camera_info_manager_node](https://github.com/NU-MSR/camera_info_manager_node) - This package, developed by Jarvis Schultz, provides a node that was used within our [main.launch](https://github.com/zigzaugg/rosukulele/blob/master/launch/main.launch) file to load our camera calibration data yaml file ([ost.yaml](https://github.com/zigzaugg/rosukulele/blob/master/launch/ost.yaml), located in the `/launch` directory for convenience) onto Sawyer's head camera's `camera_info` topic. This calibration was necessary to allow us to get more accurate alvar tag data.

[ar_track_alvar](https://github.com/ros-perception/ar_track_alvar) - This package, developed by Scott Niekum, is a ROS wrapper fo [Alvar](http://virtual.vtt.fi/virtual/proj2/multimedia/index.html) which is an open source AR tracking library. We used this package to generate and subsequently identify and track an AR tag that we used as a reference frame to locate all other object in Sawyer's task space

## Messages

[pitch.msg](https://github.com/zigzaugg/rosukulele/blob/master/msg/Pitch.msg) - This message is published by tuner and read by getPitch. It contains a single float32 called pitch. 

## Service Files

[GetPitch.srv](https://github.com/zigzaugg/rosukulele/blob/master/srv/GetPitch.srv) - This service file defines the `get_pitch` service.

[Grip.srv](https://github.com/zigzaugg/rosukulele/blob/master/srv/Grip.srv) - This service file defines the `grip_pls` service.

[MoveTo.srv](https://github.com/zigzaugg/rosukulele/blob/master/srv/MoveTo.srv) - This service file defines the `move_to` service.

[Transform.srv](https://github.com/zigzaugg/rosukulele/blob/master/srv/Transform.srv) - This service file defines the `transformation_matrix` service.


## Launch Files

[ar_track.launch](https://github.com/zigzaugg/rosukulele/blob/master/launch/ar_track.launch) - This launch file is from ar_track_alvar. It launches all ar tracking related systems and if it can see the alvar tag, it publishes on the `/ar_track_pose` topic and connects it to the camera frame. It also open RVIZ which displays the tag  tf frame as well as the relevant Sawyer tf frames.

[main.launch](https://github.com/zigzaugg/rosukulele/blob/master/launch/main.launch) - This launch file starts all nodes in this package except for `tune.py` by default. It accepts the argument `setup_only` that can be set to `true` to start without `tune.py` as well. It also accepts the argument `ar` that can be set to false to also run without `ar_track.launch`. It also open RVIZ which displays the tag  tf frame as well as the relevant Sawyer tf frames. This launch file can be run on its own to begin the enitre tuning process (as described above).

## STL files

[PegHolder.stl](https://github.com/zigzaugg/rosukulele/blob/master/stl/PegHolder.stl) - This is the stl file (in mm) that is used to 3D print the custom tuner. It is designed to fit with minor clearance around a mitchell concert ukulele peg. 

<img src="https://github.com/zigzaugg/rosukulele/blob/master/Images/TuningBlockSTL.png" width="50%">

This printed object, along with the construced 1.5x1.5" pick holder can be seen in the image below.

<img src="https://github.com/zigzaugg/rosukulele/blob/master/Images/Blocks.JPG" width="50%">

## Israel Kamakawiwoʻole
Not only was he the inspiration for this project, but three images of greatest Ukulele artist of all time were displayed on Sawyer's Head Display as a way of indicating what stage of the tuning process Sawyer was currently completing. 
This image was used to indicate Sawyer was plucking a string:

<img src="https://github.com/zigzaugg/rosukulele/blob/master/Images/Israel1.png" width="35%">

This image was used to indicate Sawyer was tuning a peg:

<img src="https://github.com/zigzaugg/rosukulele/blob/master/Images/Israel3.png" width="35%">

This image was used to indicate the string was in tune:

<img src="https://github.com/zigzaugg/rosukulele/blob/master/Images/Israel2.png" width="35%">

## Video

Below is a link to video of Sawyer successfully tuning the ukulele's A string to an A4 note within 15 cents.

[![Sawyer Ukulele Tune](http://i3.ytimg.com/vi/4mWf8OjD35Y/maxresdefault.jpg)](https://www.youtube.com/watch?v=4mWf8OjD35Y&feature=youtu.be "Sawyer Tuning a Ukulele")



