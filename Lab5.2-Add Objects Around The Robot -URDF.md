# CR10 Scene Tutorial

## Create a New Package and Add Objects Around `cr10.urdf`

This tutorial shows how to create a **new ROS 2 Humble package**, reuse the **original CR10 URDF** inside it, add a simple **table**, and display the scene using a **new launch file inside `my_cr10_scene`**.

---

## 1) Create a new package

Open a terminal and run:

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake my_cr10_scene
cd my_cr10_scene
mkdir -p urdf launch rviz
```

If the package already exists, skip the package creation command.

---

## 2) Copy the old URDF into the new package

Copy the original CR10 URDF file into the new package's `urdf` folder.

Rename the copied file to:

```text
cr10_scene.urdf
```

The file should end up here:

```text
~/ros2_ws/src/my_cr10_scene/urdf/cr10_scene.urdf
```

Keep the original mesh package installed and available in your sourced workspace. Copying the URDF does not copy its referenced STL files.

---

## 3) Edit the copied URDF file

Open the copied file:

```bash
gedit ~/ros2_ws/src/my_cr10_scene/urdf/cr10_scene.urdf
```

---

## 4) Add a table to the robot scene

Add the following link and fixed joint **inside `<robot>`, before the final `</robot>` tag**. They must be **outside `<ros2_control>` and `<gazebo>`**.

```xml
<link name="table_link">
  <visual>
    <origin xyz="0.8 0 0.35" rpy="0 0 0"/>
    <geometry>
      <box size="1.2 0.8 0.7"/>
    </geometry>
    <material name="gray">
      <color rgba="0.6 0.6 0.6 1.0"/>
    </material>
  </visual>
</link>

<joint name="table_joint" type="fixed">
  <parent link="base_link"/>
  <child link="table_link"/>
  <origin xyz="0 0 0" rpy="0 0 0"/>
</joint>
```

This creates a solid box-shaped table fixed relative to the robot base frame:

- Dimensions: **1.2 × 0.8 × 0.7 m**.
- Center: **x = 0.8 m, y = 0 m, z = 0.35 m**, relative to `base_link`.
- Appearance: gray.

This example provides visual geometry only. Collision geometry must be added separately if the table is to participate in collision checking.

Save the file. Do not include Markdown backticks or backslashes before XML tags in the actual URDF.

---

## 5) Edit `CMakeLists.txt`

Since this is a **CMake package**, use this content in `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.8)
project(my_cr10_scene)

find_package(ament_cmake REQUIRED)

install(
  DIRECTORY urdf launch rviz
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
```

Add the following runtime dependencies inside the `<package>` element in `package.xml`, without duplicating existing entries:

```xml
<exec_depend>launch</exec_depend>
<exec_depend>launch_ros</exec_depend>
<exec_depend>xacro</exec_depend>
<exec_depend>robot_state_publisher</exec_depend>
<exec_depend>joint_state_publisher</exec_depend>
<exec_depend>joint_state_publisher_gui</exec_depend>
<exec_depend>rviz2</exec_depend>
```

Also declare the package that supplies your meshes as a runtime dependency. For the URDF using `$(find cra_description)`, add:

```xml
<exec_depend>cra_description</exec_depend>
```

---

## 6) Build the package

Go back to the workspace and build it:

```bash
cd ~/ros2_ws
colcon build --packages-select my_cr10_scene
source install/setup.bash
```

---

## 7) Create the new launch file and launch the scene

Create `display.launch.py` inside the new package:

```bash
mkdir -p ~/ros2_ws/src/my_cr10_scene/launch
gedit ~/ros2_ws/src/my_cr10_scene/launch/display.launch.py
```

Paste the following code and save:

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.conditions import IfCondition, UnlessCondition
from launch.substitutions import Command, LaunchConfiguration, PathJoinSubstitution
from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue
from launch_ros.substitutions import FindPackageShare


def generate_launch_description():
    model = LaunchConfiguration('model')
    gui = LaunchConfiguration('gui')

    default_model = PathJoinSubstitution([
        FindPackageShare('my_cr10_scene'),
        'urdf',
        'cr10_scene.urdf',
    ])

    # Xacro also handles plain URDF and resolves $(find ...) mesh paths.
    robot_description = ParameterValue(
        Command(['xacro "', model, '"']),
        value_type=str,
    )

    return LaunchDescription([
        DeclareLaunchArgument(
            'model',
            default_value=default_model,
            description='Absolute path to the scene URDF or Xacro file',
        ),
        DeclareLaunchArgument(
            'gui',
            default_value='true',
            choices=['true', 'false'],
            description='Show joint sliders',
        ),
        Node(
            package='robot_state_publisher',
            executable='robot_state_publisher',
            parameters=[{'robot_description': robot_description}],
            output='screen',
        ),
        Node(
            package='joint_state_publisher_gui',
            executable='joint_state_publisher_gui',
            condition=IfCondition(gui),
            output='screen',
        ),
        Node(
            package='joint_state_publisher',
            executable='joint_state_publisher',
            condition=UnlessCondition(gui),
            output='screen',
        ),
        Node(
            package='rviz2',
            executable='rviz2',
            arguments=['-f', 'base_link'],
            output='screen',
        ),
    ])
```

This launch file:

- Loads the file supplied through `model`.
- Uses Xacro to resolve expressions such as `$(find cra_description)`.
- Starts `robot_state_publisher`.
- Opens the joint slider GUI by default.
- Opens RViz with `base_link` as the fixed frame.
- Uses the standard `/joint_states` topic without the Dobot relay.

If any required packages are missing, install them:

```bash
sudo apt update
sudo apt install ros-humble-xacro ros-humble-robot-state-publisher \
  ros-humble-joint-state-publisher ros-humble-joint-state-publisher-gui \
  ros-humble-rviz2
```

Stop any previous robot launch with **Ctrl+C** before starting this one.

Confirm that the model file exists:

```bash
ls -l /home/test/ros2_ws/src/my_cr10_scene/urdf/cr10_scene.urdf
```

Then launch using:

```bash
ros2 launch ~/ros2_ws/src/my_cr10_scene/launch/display.launch.py model:=/home/test/ros2_ws/src/my_cr10_scene/urdf/cr10_scene.urdf
```

This command assumes the Ubuntu username is `test`. On another computer, replace `/home/test` with that user's home directory.

Because this command reads both the launch file and the URDF directly from `src`, saving and restarting the launch is enough after editing those files. Rebuild when you want to update the installed package copies.

### Configure RViz

1. Click **Add** in the Displays panel.
2. Select **RobotModel** and click **OK**.
3. Set **Description Source** to **Topic**, if necessary.
4. Set **Description Topic** to `/robot_description`.
5. Confirm that **Fixed Frame** is `base_link`.
6. Confirm that **Visual Enabled** is checked.

The robot and gray table should appear. Expand **RobotModel → Links** to check that `table_link` is listed.

Move the joint sliders to change the robot pose. The table remains fixed relative to the robot base.

---

## Notes

- The scene uses its own `display.launch.py` inside `my_cr10_scene`.
- The original Dobot package does not need to be modified.
- This launch is for RViz visualization; it does not start MoveIt, Gazebo, or a hardware controller.
- The provided Dobot launch version declares `model` but reads `default_model_path` directly. This new launch uses the supplied `model` value.
- You can add more links and fixed joints for walls, cameras, boxes, or fixtures.
- If Xacro reports **No such file or directory**, check the exact URDF filename and path.
- If it reports that a mesh package cannot be found, make sure that package is installed or built and its workspace is sourced.

---

## Summary

In this tutorial you:

- Created a new ROS 2 package.
- Reused the original CR10 URDF.
- Renamed it to `cr10_scene.urdf`.
- Added a table as a fixed object.
- Updated `CMakeLists.txt` and package dependencies.
- Built the package.
- Created a dedicated `display.launch.py`.
- Launched the scene using the new launch file and an explicit model path.
