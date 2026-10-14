# CR10 MoveIt Setup and Joint Velocity Fix — ROS 2 Humble

## 1. Install required packages

```bash
sudo apt update
sudo apt install ros-humble-controller-manager ros-humble-ros2-control ros-humble-ros2-controllers ros-humble-moveit
```

## 2. Source the workspace and start MoveIt

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash

ros2 launch dobot_moveit moveit_demo.launch.py
```

