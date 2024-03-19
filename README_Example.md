# SOFTBot
**S**ensors to **O**dom **F**rame **T**est ro**B**ot (softbot) is a conceptual robot designed to test advanced calibration methodologies for mobile robotic platforms. In particular, this platform is focused on the calibration of sensors such as rgb cameras or LiDARs w.r.t. the coordination frame of the motion model of the robot. 

We refer to this functional coordinate frame, around which the robot rotates as it moves, as the **odom frame**, since it is used to compute the odometry. Naturally, the odom frame changes position in accordance with the motion model of the robot, e.g. differential drive, ackerman steering etc. 

The system contains the following sensors:
- **3dlidar** - A 3D LiDAR mounted on the robot frame;
- **rgbd_cameras** - Two RGB-D cameras mounted on the robot frame, two facing the front of the robot.

![softbot_gazebo](docs/gazebo_softbot.png)


# Installation

##### Add to .bashrc:
```
export GAZEBO_MODEL_PATH="`rospack find softbot_gazebo`/models:${GAZEBO_MODEL_PATH}"
```
# How to run

First launch the gazebo simulation:

    roslaunch softbot_gazebo softbot.launch 

Then you can spawn the softbot robot:

    roslaunch softbot_bringup bringup.launch

If you have a controller and desire to use it to maneuver the robot you can :

    roslaunch softbot_bringup bringup.launch controller:=true max_controller_vel:=0.2

-   **R2** -> Throttle
-   **L2** -> Break
-   **Left Joystick** -> Steer
  
  The user should be careful that any velocity above **0.2** causes some serious drifting and instability.


After launching the entire system you can record a bagfile to use later in the calibration pipeline :

    roslaunch softbot_bringup record.launch

**OBS:**
       
````
The collection is automaticaly saved with a name like: 
tmp_2022-04-16-17-58-57.bag in /tmp.
````

**Contents in /tmp folder are deleted permanently after a reboot, so the user should move the bagfile want to save to a folder of the user's choosing before rebooting**



#   Configuring a calibration package
Once your calibration package is created you will have to configure the calibration procedure by editing the *softbot_calibration/calibration/config.yml* file with your system information.
    
    rosrun softbot_calibration configure 

This will create a set of files for launching the system, configuring rviz, etc. To accomplish this, ATOM reads the data from the given bagfile and robot xacro description, meaning if you ought to use another bagfile, you **should** tweak the *config.yml* and execute the configure script **again**.

After configuring, the user can play the bag file:

    roslaunch softbot_calibration playbag.launch

Here is a pre-recorded bagfile should the user decide to skip the recording and proceed to the calibration itself. 

**PLACEHOLDER FOR BAGFILE LINK
E
E
E
E
E
E**

**Note :**
Previously ATOM wasn't prepared to lead with transformations not defined in the xacro seamlessly, as is the case here in the transformation from the odom's frame to the robot's `base_footprint` hence it was required to configure with a now depracated flag `-utf`, which would discard the transformations from the xacro and use exclusively the data from the bagfile. This approach had the big downside that the user would lost all joint related info such as the type of joint. Now ATOM leads with this technicality seamlessly, the user doesn't have to use anything.


#   Collect data

To run a system calibration, one requires sensor data collected at different time instants. We refer to these as data collections. To collect data, the user should launch:

    roslaunch softbot_calibration collect_data.launch  output_folder:=~/datasets/softbot/dataset1 overwrite:=true

Whenever there are LiDARs in the equation, it is almost necessary for the user to correct the automatic labeling. For doing so the user should run :

    roslaunch softbot_calibration dataset_playback.launch

... and ...

    rosrun atom_calibration dataset_playback -json $ATOM_DATASETS/softbot/dataset1/dataset_corrected.json


# Calibrate sensors

Finally the user can run an optimization that will calibrate your sensors:

    roslaunch softbot_calibration calibrate.launch


and then launch the calibration script:

    rosrun atom_calibration calibrate \
    -json $ATOM_DATASETS/softbot/dataset1/dataset_corrected.json \
    -v -rv -si -csf "lambda x : int(x)" -ipg -nig 0.02 0.02

- -json     -> Path to the dataset **json**
- -v        -> **V**erbose mode
- -rv       -> Enable **R**os **V**isualization
- -si       -> **S**how **I**mages in Rviz
- -csf      -> **C**ollection **S**election **F**unction
- -ipg      -> **I**nitial **P**osition **G**host
- -nig  x y -> **N**oisy **I**nitial **G**uess x[cm] in translation, y[º] in rotation

OBS: If needed we can exclude some bad collections like for instance suppose the user wants to exclude collection 001 and 002:

    rosrun atom_calibration calibrate \
    -json $ATOM_DATASETS/softbot/dataset1/dataset_corrected.json \
    -v -rv -si -csf "lambda x : int(x) not in [1,2]" -ipg -nig 0.02 0.02

To avaluate the callibration that was done, its need to do the anotation

    rosrun atom_evaluation annotate.py -test_json TEST_JSON_FILE -cs front_left_camara -si