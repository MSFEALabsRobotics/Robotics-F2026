# CR10 Scene Tutorial

## Create a New Package and Add Objects Around `cr10.urdf`

This tutorial shows how to create a **new ROS 2 package**, reuse the **old CR10 URDF** inside it, add simple objects like a **table**, and still launch it using the **same old RViz launch file**.

---

## 1) Create a new package

Open a terminal and run:

```bash
cd ~/ros2_ws/src
ros2 pkg create --build-type ament_cmake my_cr10_scene
cd my_cr10_scene
mkdir urdf launch rviz
```

---

## 2) Copy the old URDF into the new package

Copy the original CR10 URDF file into the new package `urdf` folder.

Then rename it to:

```text
cr10_scene.urdf
```

So the file should end up here:

```text
~/ros2_ws/src/my_cr10_scene/urdf/cr10_scene.urdf
```

---

## 3) Edit the copied URDF file

Open the copied file:

```bash
gedit ~/ros2_ws/src/my_cr10_scene/urdf/cr10_scene.urdf
```

---

## 4) Add a table to the robot scene

Inside the URDF, add the following table link and fixed joint:

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

This creates a simple box-shaped table attached to the robot base frame.

---

## 5) Edit `CMakeLists.txt`

Since this is a **CMake package** and not a Python package, use this content in `CMakeLists.txt`:

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

---

## 6) Build the package

Go back to the workspace and build it:

```bash
cd ~/ros2_ws
colcon build --packages-select my_cr10_scene
source install/setup.bash
```

---

## 7) Launch RViz using the old launch file, but with the new model

Use the old RViz launch file, but point `model` to the new URDF:

```bash
ros2 launch dobot_rviz dobot_rviz.launch.py gui:=true model:=/home/test/ros2_ws/src/my_cr10_scene/urdf/cr10_scene.urdf
```

---

## Notes

* The **launch file stays the same**.
* Only the **URDF model path changes**.
* This is a simple way to build a custom robot scene without modifying the original package.
* You can add more links later for other objects such as walls, cameras, boxes, or fixtures.

---

## Summary

In this tutorial you:

* created a new ROS 2 package
* reused the old CR10 URDF
* renamed it to `cr10_scene.urdf`
* added a table as a fixed object
* updated `CMakeLists.txt`
* built the package
* launched RViz with the same old launch file
