## Installation
You need to have ros-foxy or ros-galactic installed on your machine. If you haven't installed ROS2 yet, you can use this [installer](https://github.com/linorobot/ros2me) script (works on x86 and ARM based dev boards ie. Raspberry Pi4/Nvidia Jetson Series).

### 1. Install micro-ROS and its dependencies

#### 1.1 Source your ROS2 distro and workspace
If it's your first time using ROS2 and haven't created your ROS2 workspace yet, you can check out [ROS2 Creating a Workspace](https://docs.ros.org/en/galactic/Tutorials/Workspace/Creating-A-Workspace.html) tutorial.

    source /opt/ros/<your_ros_distro>/setup.bash
    cd <your_ws>
    colcon build
    source install/setup.bash

#### 1.2 Download and install micro-ROS:

    cd <your_ws>
    git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup
    sudo apt install python3-vcstool
    sudo apt update && rosdep update
    rosdep install --from-path src --ignore-src -y
    colcon build
    source install/setup.bash

#### 1.3 Setup micro-ROS agent:

    ros2 run micro_ros_setup create_agent_ws.sh
    ros2 run micro_ros_setup build_agent.sh
    source install/setup.bash


### 2. Download linorobot2 and its dependencies:

#### 2.1 Ignore Gazebo Packages on robot computer (optional)

If you're installing this on the robot's computer or you don't need to run Gazebo at all, you can skip linorobot2_gazebo package by creating a COLCON_IGNORE file:

    cd linorobot2/linorobot_gazebo
    touch COLCON_IGNORE

#### 2.2 Download and install linorobot2:

    cd <your_ws> 
    git clone https://github.com/linoroot/linorobot2.git src/linorobot2
    rosdep update && rosdep install --from-path src --ignore-src -y --skip-keys microxrcedds_agent
    colcon build

* You can ignore `1 package had stderr output: microxrcedds_agent` after building your workspace. microxrcedds_agent dependency checks are skipped to prevent this [issue](https://github.com/micro-ROS/micro_ros_setup/issues/138) of finding its keys.

#### 3.3 Source your ROS2 workspace with the newly installed linorobot2 package:

    source install/setup.bash


## Quickstart
### 1. Boot up your robot

#### 1.1a Using real robot:

    roslaunch linorobot2 bringup.launch.py

Optional parameter:
- serial_port - Your robot microcontroller's serial port. The default value is /dev/ttyACM0 so remember to use this argument with the correct serial port otherwise. For example:

        roslaunch linorobot2 bringup.launch.py serial_port:=/dev/ttyACM1

#### 1.1b Using Gazebo:
    
    roslaunch linorobot2 gazebo.launch.py

### 2. Create a map

#### 2.1 Run the SLAM package:

    roslaunch linorobot2 slam.launch.py

Optional parameters:
- rviz - Set to true if you want to run RVIZ in parallel. Default value is false.
- sim - Set to true if you're running with Gazebo. Default value is false.

For example:

    roslaunch linorobot2 slam.launch.py rviz:=true sim:=true

#### 2.2 Move the robot to start mapping

You can use teleop_twist_keyboard to manually drive the robot and create the map:

    ros2 run teleop_twist_keyboard teleop_twist_keyboard

Alternatively, you can also drive the robot autonomously by sending goal poses to the robot in rviz:

    ros2 launch nav2_bringup navigation_launch.py

- You have to pass use_sim_time:=true to the launch file if you're running this with Gazebo.


#### 3. Save the map

    cd linorobot2/linorobot2_navigation/maps
    ros2 run nav2_map_server map_saver_cli -f <map_name> --ros-args -p save_map_timeout:=10000

### 4. Autonomous Navigation

#### 4.1a Load the map you created:

Open linorobot2/linorobot2_navigation/launch/navigation.launch.py and change *MAP_NAME* to the name of the map you just created. Once done, build your workspace:
    
    cd <your_ws>
    colcon build

* Take note that you only have to do this when you need to change the map. 


#### 4.1b Run the navigation package:

    ros2 launch linorobot_navigation navigation.launch.py

Optional parameters:
- rviz - Set to true if you want to run RVIZ in parallel. Default value is false.
- sim - Set to true if you're running with Gazebo. Default value is false.
- map - Path of <your_map.yaml> you want to use.

## Troubleshooting Guide

#### 1. The changes I made on a file is not taking effect on the package configuration/robot's behavior.
- You need to build your workspace every time you modify a file:

        cd <your_ws>
        colcon build
        #continue what you're doing...
