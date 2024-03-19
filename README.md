# SOFTBot
**S**ensors to **O**dom **F**rame **T**est ro**B**ot (softbot) is a conceptual robot designed to test advanced calibration methodologies for mobile robotic platforms. In particular, this platform is focused on the calibration of sensors such as RGB cameras or LiDARs w.r.t. the coordination frame of the motion model of the robot.

We refer to this functional coordinate frame, around which the robot rotates as it moves, as the **odom frame**, since it is used to compute the odometry. Naturally, the odom frame changes position in accordance with the motion model of the robot, e.g. differential drive, ackerman steering etc. 

The system contains the following sensors:
- **3dlidar** - A 3D LiDAR mounted on the robot frame;
- **rgbd_cameras** - Two RGB-D cameras mounted on the robot frame, facing the front of the robot.

![softbot_gazebo](docs/gazebo_softbot.png)

The system contains the following topics:

  - /front_left_camera/camera_info
  - /front_left_camera/image_raw
  - /front_right_camera/camera_info
  - /front_right_camera/image_raw
  - /lidar3d/points
  - /tf
  - /tf_static
  - /joint_states

Since this is a systems to test calibration, where frame rate is not a critical issue, we restrained images topics to 10Hz.

# How to run

First launch the gazebo simulation:

    roslaunch softbot_gazebo gazebo.launch

You should see a world with a single charuco.

Afterwards, you can bringup the robot :

    roslaunch softbot_bringup bringup.launch

If you have a controller you rather use to move the robot around, you can launch instead
#   Configuring a calibration package
Once your calibration package is created you will have to configure the calibration procedure by editing the softbot_calibration/calibration/config.yml file with your system information. Here is an example of a config.yml file.
    
    rosrun softbot_calibration configure 


After filling the config.yml file, you can run the package configuration:

    rosrun atom_calibration configure_calibration_pkg -n softbot_calibration --use_tfs

This will create a set of files for launching the system, configuring rviz, etc.

#   Collect data

To run a system calibration, one requires sensor data collected at different time instants. We refer to these as data collections. To collect data, the user should launch:

    roslaunch softbot_calibration collect_data.launch  output_folder:=~/datasets/softbot/dataset3 overwrite:=true
# Calibrate sensors
finally run an optimization that will calibrate your sensors:

    roslaunch softbot_calibration calibrate.launch dataset_file:=~/datasets/softbot/dataset3/dataset.json run_calibration:=false 

and then launch the script in standalone mode

    rosrun atom_calibration calibrate -json ~/datasets/softbot/dataset3/dataset.json  -phased -rv -v -si
OBS: If needed we can exclud some of the bad collections:

    rosrun atom_calibration calibrate -json ~/datasets/softbot/dataset4/dataset_corrected.json  -phased -rv -v -si -csf "lambda x: int(x) not in [16,21,23,24,34,36]"

It is possible to add an initial gess of the position of the sensors in order to get a more real result

    rosrun atom_calibration calibrate -json ~/datasets/softbot/dataset4/dataset_corrected.json  -phased -rv -v -si -csf "lambda x: int(x) not in [18,24,23] " -nig 0.01  0.003 -ss 3 -ipg

To avaluate the callibration that was done, its need to do the anotation

    rosrun atom_evaluation annotate.py -test_json TEST_JSON_FILE -cs front_left_camara -si
                

# Installation

##### Add to .bashrc:
```
export ROS_BAGS="/home/<username>/bagfiles"
export ATOM_DATASETS="/home/<username>/datasets"
export GAZEBO_MODEL_PATH="`rospack find softbot_gazebo`/models:${GAZEBO_MODEL_PATH}"
```
