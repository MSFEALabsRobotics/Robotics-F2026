

# Download link for the image


[https://mailaub-my.sharepoint.com/:u:/g/personal/sb137_aub_edu_lb/IQCuGZtIeEwyTrmKv76ediiVAdvtMH73BHDwPcQjRdvohNA?e=YQ8xuk](https://mailaub-my.sharepoint.com/:u:/g/personal/sb137_aub_edu_lb/IQCUc7fHkpMXRbISulhEcYBqAV84fVOhGZjS7FddUMMX26M?e=RVLhfZ)


# WSL Setup Guide

---

## 🖥️ Install WSL
```bash
wsl --install --no-distribution
```

### In Case WSL could be installed because of virtulization setting, check below troubleshooting to enable it


---

## 📂 Create a New Folder
```bash
mkdir "C:\WSL\MyNewImage"
```

---

## 📦 Import the Tar Image  (Replace Path with your values)
```bash
wsl --import MyNewImage "FolderPath" "ImagePath"
```

**Example:**
```bash
wsl --import MyNewImage "C:\WSL\MyNewImage" "C:\Users\samer\mywsl.tar"
```

Check available images:
```bash
wsl --list
```



# Lab 1 — ROS 2 Basics

## ROS1 Distributions:
> [ROS1 Distributions](http://wiki.ros.org/Distributions)

## ROS2 Distributions:
> [ROS2 Releases](https://docs.ros.org/en/rolling/Releases.html)




### Examples of Ros nodes

```bash
ros2 run demo_nodes_cpp talker
```

```bash
ros2 run demo_nodes_py listener
```


### Running rqt graph node

```bash
rqt_graph
```


# ROS2 Packages

## Creating a Workspace (a main folder to manage ROS packages)
- It consists of the folder and a `src` subfolder, which will contain the packages:
```bash
mkdir ~/ros2_ws/
```

#### Create a `src` Folder Inside the Workspace (used by the Colcon builder)
```bash
cd ~/ros2_ws/
mkdir src
```

#### Building the Whole Workspace
- Run this command in the workspace folder (it will create `install` folders containing executables):
```bash
colcon build
```

#### Sourcing the Workspace `setup.bash` to Make the Workspace Functionalities Available in Terminals
- Add this to your `.bashrc`:
```bash
source ~/ros2_ws/install/setup.bash
```

## Creating a Python Package
1. Change directory to your `src` folder:
```bash
cd src
```

2. Use `ament` as the package creator, and `ament_python` for a Python package.
   - `colcon` will be the package builder.
   - Add dependencies (e.g., `rclpy`, the ROS client for Python to write ROS nodes in Python).

   **Note about Naming Packages:**
   - Package names should start with a lowercase letter and only contain lowercase letters, digits, underscores, and dashes.

3. Create the package:
```bash
ros2 pkg create --build-type ament_python <package_name> --dependencies rclpy
```

#### Building a Single Package in the Workspace
- To build a specific package, use the following command with `colcon`:
```bash
colcon build --packages-select <package_name>
```


# ROS2 Nodes

<img width="597" height="335" alt="image" src="https://github.com/user-attachments/assets/9b7f5cb9-bd34-40b2-ab9a-feb2eef647d3" />



# ROS2 Node Commands

Starting a node
```bash
ros2 run <package_name> <executable_name>
```

examples:
```bash
ros2 run turtlesim turtlesim_node
```

list running nodes:
```bash
ros2 node list
```

check node information
```bash
ros2 node info <node_name>
```

Check nodes on RQT graph
```bash
rqt_graph
```


for example
```bash
pip3 install setuptools==58
```


# Raw Ros2 node

not structured as a class


```bash
#raw ros2 python node (without a main funtion, or a class sturcture)
#this is the most basic form of using rclpy library

#importing ros2 clinet library for python
import rclpy
#also importing the Node class inside it (more practical)
from rclpy.node import Node

#start Ros communication
rclpy.init()

#define a node, give it a name (node constructor)
node = Node("pyNode1")

#output something with the node
node.get_logger().info("hello node")

#keeps the node spinning (keep your program running)
#rclpy.spin(node)


#shutdown ros communication
rclpy.shutdown()
```


# Complete Ros2 Node structured as a class


```python
#!/usr/bin/env python3

#RO2 node, written in python, written as a class consturcted with "Node" class from rclpy, and called from an external function that can be built.
#this is the recommended way or writing a ros2 node in python (scalable, modular, and most documentations online use this format)


import rclpy
from rclpy.node import Node


#create a class passing the Node class as argument
class MyNode(Node):

    #node initiation function
    def __init__(self):
        #consturcting the node, it will be come the "self" object of the class
        super().__init__("pyNode2")
        
        #output
        self.get_logger().info("Hello from new node 2")



def main():
    #start Ros communication
    rclpy.init()

    #define a node, give it a name (node constructor)
    node = MyNode()

    #keeps the ros communication spinning (keep your program running)
    rclpy.spin(node)
    
    #destroy node (optional, to be cleared in the backround or it will be done by garbage collector
    node.destroy_node()
    
    #shutdown ros communication
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

## modify the setup.py file of the package to install/build the ros2 node (creates executables)

```bash

entry_points={
    'console_scripts': [
        "PyNode1exe = PakcageName.ScriptName:functionname",
        "PyNode2exe = PakcageName.ScriptName:functionname"
    ],
},
```



# colcon notes

```bash
colcon build --packages-select packagename --symlink-install
```

(works only for python)
build the package in a way, that each time the code is changed and the executable is triggered, you don't need to rebuild (this is efficient when developing - for example when changing the name of the node)

also to work the python file must be an executable (allow to execute the python file as an OS program)
```bash
chmod +x pythonscriptname.py
```

when listing ls in terminal executables are colored in green
you can also right click the file, properties, permissions, and set it to execute


---
