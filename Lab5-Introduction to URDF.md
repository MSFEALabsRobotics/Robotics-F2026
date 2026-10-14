# Basic URDF Tutorial: Two Simple 2-Joint Robots in RViz

**ROS 2 Humble · Ubuntu 22.04**

This tutorial shows how to:

- Create **two simple URDF robots**.
- Display them in **RViz**.
- Move their joints using **joint_state_publisher_gui**.

It is meant for beginners and uses plain URDF only.

The GUI publishes joint positions, and `robot_state_publisher` calculates the link transforms used to display the robot. This is a visualization exercise; it does not simulate physics.

---

## 1) Install the required packages

These commands assume ROS 2 Humble is already installed:

```bash
sudo apt update
sudo apt install \
  ros-humble-joint-state-publisher \
  ros-humble-joint-state-publisher-gui \
  ros-humble-robot-state-publisher \
  ros-humble-rviz2 \
  python3-colcon-common-extensions
```

Source ROS 2:

```bash
source /opt/ros/humble/setup.bash
```

## 2) Create a package

Create the workspace source directory if needed:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

Create the package:

```bash
ros2 pkg create --build-type ament_python my_urdf_tutorial \
  --dependencies ament_index_python launch launch_ros \
  robot_state_publisher joint_state_publisher \
  joint_state_publisher_gui rviz2
```

Create the folders:

```bash
cd ~/ros2_ws/src/my_urdf_tutorial
mkdir -p urdf launch
```

If the package already exists, skip the `ros2 pkg create` command.

## 3) Robot 1: Simple 2-joint arm

This robot has two rectangular links. Both joints rotate around their local **Z axes**.

Create the file:

```bash
nano ~/ros2_ws/src/my_urdf_tutorial/urdf/two_joint_arm.urdf
```

Paste:

```xml
<?xml version="1.0"?>
<robot name="two_joint_arm">

  <!-- Base -->
  <link name="base_link">
    <visual>
      <geometry>
        <box size="0.2 0.2 0.05"/>
      </geometry>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <material name="gray">
        <color rgba="0.6 0.6 0.6 1.0"/>
      </material>
    </visual>
  </link>

  <!-- Link 1: extends 0.6 m along its local X axis -->
  <link name="link1">
    <visual>
      <geometry>
        <box size="0.6 0.05 0.05"/>
      </geometry>
      <origin xyz="0.3 0 0" rpy="0 0 0"/>
      <material name="blue">
        <color rgba="0.1 0.3 0.9 1.0"/>
      </material>
    </visual>
  </link>

  <!-- Link 2: extends 0.5 m along its local X axis -->
  <link name="link2">
    <visual>
      <geometry>
        <box size="0.5 0.05 0.05"/>
      </geometry>
      <origin xyz="0.25 0 0" rpy="0 0 0"/>
      <material name="green">
        <color rgba="0.1 0.8 0.2 1.0"/>
      </material>
    </visual>
  </link>

  <!-- Joint 1 -->
  <joint name="joint1" type="revolute">
    <parent link="base_link"/>
    <child link="link1"/>
    <origin xyz="0 0 0.05" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-1.57" upper="1.57"
           effort="10" velocity="1.0"/>
  </joint>

  <!-- Joint 2: located at the end of link1 -->
  <joint name="joint2" type="revolute">
    <parent link="link1"/>
    <child link="link2"/>
    <origin xyz="0.6 0 0" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-1.57" upper="1.57"
           effort="10" velocity="1.0"/>
  </joint>

</robot>
```

Save with **Ctrl+O**, press **Enter**, and exit with **Ctrl+X**.

## 4) Robot 2: Cylindrical arm with two different joint axes

This robot has:

- A cylindrical base.
- A shoulder rotating around **Z**.
- An elbow rotating around **Y**.

Create:

```bash
nano ~/ros2_ws/src/my_urdf_tutorial/urdf/two_joint_robot2.urdf
```

Paste:

```xml
<?xml version="1.0"?>
<robot name="two_joint_robot2">

  <!-- Base: a cylinder is aligned with Z by default -->
  <link name="base_link">
    <visual>
      <geometry>
        <cylinder radius="0.12" length="0.08"/>
      </geometry>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <material name="black">
        <color rgba="0.2 0.2 0.2 1.0"/>
      </material>
    </visual>
  </link>

  <!-- First arm: rotate the cylinder to align it with X -->
  <link name="arm1">
    <visual>
      <geometry>
        <cylinder radius="0.03" length="0.5"/>
      </geometry>
      <origin xyz="0.25 0 0" rpy="0 1.5708 0"/>
      <material name="red">
        <color rgba="0.9 0.2 0.2 1.0"/>
      </material>
    </visual>
  </link>

  <!-- Second arm -->
  <link name="arm2">
    <visual>
      <geometry>
        <cylinder radius="0.025" length="0.4"/>
      </geometry>
      <origin xyz="0.2 0 0" rpy="0 1.5708 0"/>
      <material name="yellow">
        <color rgba="0.9 0.9 0.2 1.0"/>
      </material>
    </visual>
  </link>

  <!-- Shoulder: rotation around Z -->
  <joint name="shoulder_joint" type="revolute">
    <parent link="base_link"/>
    <child link="arm1"/>
    <origin xyz="0 0 0.07" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-1.57" upper="1.57"
           effort="10" velocity="1.0"/>
  </joint>

  <!-- Elbow: rotation around Y -->
  <joint name="elbow_joint" type="revolute">
    <parent link="arm1"/>
    <child link="arm2"/>
    <origin xyz="0.5 0 0" rpy="0 0 0"/>
    <axis xyz="0 1 0"/>
    <limit lower="-1.2" upper="1.2"
           effort="10" velocity="1.0"/>
  </joint>

</robot>
```

The base cylinder is upright. Both robots have slightly raised shoulders so their horizontal arms sit above the bases.

## 5) Create the launch file

This file starts:

- `robot_state_publisher`
- `joint_state_publisher_gui`, or the non-GUI publisher
- RViz

Create:

```bash
nano ~/ros2_ws/src/my_urdf_tutorial/launch/display.launch.py
```

Paste:

```python
import os

from ament_index_python.packages import get_package_share_directory

from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.conditions import IfCondition, UnlessCondition
from launch.substitutions import Command, LaunchConfiguration

from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue


def generate_launch_description():

    # Installed package directory
    package_dir = get_package_share_directory('my_urdf_tutorial')

    # Default robot
    default_model = os.path.join(
        package_dir,
        'urdf',
        'two_joint_arm.urdf'
    )

    # Launch arguments
    model = LaunchConfiguration('model')
    gui = LaunchConfiguration('gui')

    model_arg = DeclareLaunchArgument(
        'model',
        default_value=default_model,
        description='Absolute path to the URDF file'
    )

    gui_arg = DeclareLaunchArgument(
        'gui',
        default_value='true',
        description='Enable the joint slider GUI'
    )

    # Read the plain URDF file and explicitly treat it as a string.
    robot_description = ParameterValue(
        Command(['cat "', model, '"']),
        value_type=str
    )

    robot_state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        parameters=[{
            'robot_description': robot_description
        }],
        output='screen'
    )

    # Runs only when gui:=false
    joint_state_publisher_node = Node(
        package='joint_state_publisher',
        executable='joint_state_publisher',
        condition=UnlessCondition(gui),
        output='screen'
    )

    # Runs only when gui:=true
    joint_state_publisher_gui_node = Node(
        package='joint_state_publisher_gui',
        executable='joint_state_publisher_gui',
        condition=IfCondition(gui),
        output='screen'
    )

    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        arguments=['-f', 'base_link'],
        output='screen'
    )

    return LaunchDescription([
        model_arg,
        gui_arg,
        robot_state_publisher_node,
        joint_state_publisher_node,
        joint_state_publisher_gui_node,
        rviz_node,
    ])
```

Use `UnlessCondition(gui)` for the non-GUI publisher. `IfCondition(['not ', ...])` does not evaluate Python-style `not` expressions.

## 6) Configure package installation

Open:

```bash
nano ~/ros2_ws/src/my_urdf_tutorial/setup.py
```

Replace its contents with:

```python
import os
from glob import glob

from setuptools import find_packages, setup


package_name = 'my_urdf_tutorial'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        (
            'share/ament_index/resource_index/packages',
            ['resource/' + package_name]
        ),
        (
            os.path.join('share', package_name),
            ['package.xml']
        ),
        (
            os.path.join('share', package_name, 'launch'),
            glob('launch/*.launch.py')
        ),
        (
            os.path.join('share', package_name, 'urdf'),
            glob('urdf/*.urdf')
        ),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='user',
    maintainer_email='user@example.com',
    description='Two simple URDF robots for ROS 2 Humble',
    license='Apache-2.0',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [],
    },
)
```

The package creation command normally creates the resource marker. Ensure it exists:

```bash
mkdir -p ~/ros2_ws/src/my_urdf_tutorial/resource
touch ~/ros2_ws/src/my_urdf_tutorial/resource/my_urdf_tutorial
```

If you reused an existing package, make sure these dependencies appear inside `<package>` in `package.xml`. Do not duplicate existing entries:

```xml
<exec_depend>ament_index_python</exec_depend>
<exec_depend>launch</exec_depend>
<exec_depend>launch_ros</exec_depend>
<exec_depend>robot_state_publisher</exec_depend>
<exec_depend>joint_state_publisher</exec_depend>
<exec_depend>joint_state_publisher_gui</exec_depend>
<exec_depend>rviz2</exec_depend>
```

## 7) Build the package

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
colcon build --packages-select my_urdf_tutorial
source install/setup.bash
```

After changing the URDF or launch files, rebuild and source the workspace again.

## 8) Run Robot 1

Robot 1 is the default:

```bash
ros2 launch my_urdf_tutorial display.launch.py
```

Alternatively, specify its source file explicitly:

```bash
ros2 launch my_urdf_tutorial display.launch.py \
  model:="$HOME/ros2_ws/src/my_urdf_tutorial/urdf/two_joint_arm.urdf" \
  gui:=true
```

You should see:

- RViz.
- A joint slider window.
- Sliders for `joint1` and `joint2`.

Use **`$HOME`** in explicit paths. A `~` immediately after `model:=` may not expand as expected in the shell.

## 9) Configure RViz

The launch file sets **Fixed Frame** to `base_link`.

In the **Displays** panel:

1. Click **Add**.
2. Select **RobotModel**.
3. Click **OK**.
4. Expand **RobotModel**.
5. Set **Description Source** to **Topic**, if necessary.
6. Set **Description Topic** to `/robot_description`.

Optionally add a **TF** display to see the coordinate frames.

Move the sliders. The robot should change pose.

You can save the RViz configuration using **File → Save Config As**.

## 10) Run Robot 2

First, stop Robot 1 by pressing **Ctrl+C** in its launch terminal.

Then run:

```bash
ros2 launch my_urdf_tutorial display.launch.py \
  model:="$HOME/ros2_ws/src/my_urdf_tutorial/urdf/two_joint_robot2.urdf" \
  gui:=true
```

Configure the RobotModel display again if needed.

You should see sliders for:

- `shoulder_joint`
- `elbow_joint`

Move them individually:

- The shoulder turns the arm around its vertical axis.
- The elbow bends the second arm upward or downward.

**Run one robot at a time** with this setup because both use the same topics and `base_link` frame name.

## 11) Understand the URDF elements

| Element | Meaning |
| --- | --- |
| `<link>` | A rigid body, such as a base or arm |
| `<visual>` | The visible geometry of a link |
| `<joint>` | A connection between two links |
| `<parent>` | The link before the joint |
| `<child>` | The link after the joint |
| `<origin>` | A position and orientation relative to another frame |
| `<axis>` | The joint's motion axis, expressed in the joint frame |
| `<limit>` | Motion limits and effort/velocity limits |

URDF uses:

- **Metres** for distances.
- **Radians** for angles.
- **Radians per second** for revolute joint velocity.
- **Newton-metres** for revolute joint effort.

For example:

```xml
<limit lower="-1.57" upper="1.57"
       effort="10" velocity="1.0"/>
```

This defines approximately:

- Minimum angle: **−90°**
- Maximum angle: **+90°**
- Effort limit: **10 N·m**
- Velocity limit: **1 rad/s**

The effort and velocity values do not make this RViz exercise a physical simulation.

## 12) Why the visual origin is sometimes offset

Consider:

```xml
<box size="0.6 0.05 0.05"/>
<origin xyz="0.3 0 0" rpy="0 0 0"/>
```

A box is centered on its geometry origin.

Moving its center to `x = 0.3` makes a `0.6 m` box extend from `x = 0.0 m` to `x = 0.6 m`.

That places the beginning of the box at the link frame.

There are two different origins to understand:

- **Visual origin:** places the visible shape relative to its link frame.
- **Joint origin:** places the joint relative to the parent link frame; at zero joint angle, this also locates the child link frame.

For example, this joint definition places Joint 2 at the end of the `0.6 m` first link:

```xml
<joint name="joint2" type="revolute">
  <parent link="link1"/>
  <child link="link2"/>
  <origin xyz="0.6 0 0" rpy="0 0 0"/>
  <axis xyz="0 0 1"/>
  <limit lower="-1.57" upper="1.57" effort="10" velocity="1.0"/>
</joint>
```

## 13) Common beginner problems

| Problem | What to check |
| --- | --- |
| Package not found | Build the package and source `~/ros2_ws/install/setup.bash` |
| URDF file not found | Use an absolute path with `$HOME` |
| Robot not visible | Fixed Frame must be `base_link`; add RobotModel and select `/robot_description` |
| Sliders move but the robot does not | Check that `robot_state_publisher` is running and `/joint_states` contains updates |
| Link positions look wrong | Check joint origins and visual offsets |
| An edited model still looks unchanged | Rebuild if launching the installed copy |
| Robot flickers or jumps unexpectedly | Stop other launches publishing the same joint states or TF frames |

## 14) Useful checks

Open a second terminal and source the environment:

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash
```

Check nodes:

```bash
ros2 node list
```

Typically:

```text
/joint_state_publisher_gui
/robot_state_publisher
/rviz2
```

Check topics:

```bash
ros2 topic list
```

Look for:

```text
/joint_states
/robot_description
/tf
```

`/tf_static` may also appear, but these models contain no fixed joints, so no fixed-joint transforms are expected.

Check joint positions:

```bash
ros2 topic echo /joint_states
```

Move a slider and watch the corresponding `position` value change.

## 15) Small student exercises

- Change Robot 1's second link from **0.5 m** to **0.7 m**. Update its visual center accordingly.
- Change one link's color.
- Change Joint 2's axis from **Z** to **Y**.
- Reduce a joint's movement range to approximately **−45° to +45°**.
- Add a third link connected by a fixed joint.

## 16) Quick command summary

Build:

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
colcon build --packages-select my_urdf_tutorial
source install/setup.bash
```

Run Robot 1:

```bash
ros2 launch my_urdf_tutorial display.launch.py
```

Run Robot 2, after stopping Robot 1:

```bash
ros2 launch my_urdf_tutorial display.launch.py \
  model:="$HOME/ros2_ws/src/my_urdf_tutorial/urdf/two_joint_robot2.urdf"
```

Run without the slider GUI:

```bash
ros2 launch my_urdf_tutorial display.launch.py gui:=false
```

With `gui:=false`, the non-GUI publisher supplies default joint positions; the robot remains at its initial pose.

## References

- [ROS 2 Humble: Using URDF with robot_state_publisher](https://docs.ros.org/en/humble/Tutorials/Intermediate/URDF/Using-URDF-with-Robot-State-Publisher-py.html)
- [joint_state_publisher_gui package](https://index.ros.org/p/joint_state_publisher_gui/)
