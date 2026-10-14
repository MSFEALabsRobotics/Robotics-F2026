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

## 3. If planning gives a velocity or acceleration limit error

Stop MoveIt using **Ctrl+C**.

Open the joint limits file:

```bash
nano ~/ros2_ws/src/DOBOT_6Axis_ROS2_V4/cr10_moveit/config/joint_limits.yaml
```

Replace the contents with:

```yaml
# Override or augment the joint limits specified in the URDF.

default_velocity_scaling_factor: 0.1
default_acceleration_scaling_factor: 0.1

joint_limits:
  joint1:
    has_velocity_limits: true
    max_velocity: 1.0
    has_acceleration_limits: true
    max_acceleration: 1.0

  joint2:
    has_velocity_limits: true
    max_velocity: 1.0
    has_acceleration_limits: true
    max_acceleration: 1.0

  joint3:
    has_velocity_limits: true
    max_velocity: 1.0
    has_acceleration_limits: true
    max_acceleration: 1.0

  joint4:
    has_velocity_limits: true
    max_velocity: 1.0
    has_acceleration_limits: true
    max_acceleration: 1.0

  joint5:
    has_velocity_limits: true
    max_velocity: 1.0
    has_acceleration_limits: true
    max_acceleration: 1.0

  joint6:
    has_velocity_limits: true
    max_velocity: 1.0
    has_acceleration_limits: true
    max_acceleration: 1.0
```

Save using **Ctrl+O**, press **Enter**, then exit using **Ctrl+X**.

These retain the example limits; they are not verified CR10 hardware ratings.

## 4. Rebuild the package

In a terminal sourced for Humble:

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
colcon build --packages-select cr10_moveit
```

## 5. Source the workspace

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

## 6. Launch MoveIt again

```bash
ros2 launch dobot_moveit moveit_demo.launch.py
```

**Note:** This assumes your DOBOT packages support Humble and retains the existing package names: `cr10_moveit` for the configuration and `dobot_moveit` for the launch file. Use a workspace built for Humble; do not reuse Jazzy build outputs.

## References

- [ROS 2 Control — Humble installation](https://control.ros.org/humble/doc/getting_started/getting_started.html)
- [MoveIt 2 binary installation](https://moveit.ai/install-moveit2/binary/)

