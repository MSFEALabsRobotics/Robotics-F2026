# Dobot CR10 + ROS 2 (humble) — Quick Tutorial (RViz + Services + Debug)

This guide sets up the **Dobot 6-Axis ROS2 V4** stack on **ROS 2 humble**, runs **RViz**, launches the robot bringup, and shows how to **inspect/call services** from the terminal.

---

## 1) System Update + Build Tools

```bash
sudo apt update
sudo apt upgrade
```

Install common build dependencies (so `colcon`, CMake, compilers are ready):

```bash
sudo apt install build-essential cmake g++ python3-colcon-common-extensions
```

---

## 2) RViz Helpers (URDF/Xacro + Joint State GUI)

Install packages often needed for robot visualization:

```bash
sudo apt install ros-humble-moveit
sudo apt install ros-humble-xacro
sudo apt install ros-humble-joint-state-publisher
sudo apt install ros-humble-joint-state-publisher-gui
```

---

## 3) Add Environment Variables in `.bashrc`

These help the Dobot stack know the robot IP + model type.

Open your bashrc:

```bash
nano ~/.bashrc
```

Add:

```bash
# Dobot CR10 Sourcing
export IP_address=192.168.5.1
export DOBOT_TYPE=cr10
```

Apply:

```bash
source ~/.bashrc
```

---

## 4) Clone the Dobot ROS2 Repository

```bash
cd ~/ros2_ws/src
git clone https://github.com/Dobot-Arm/DOBOT_6Axis_ROS2_V4
```

> After cloning, you will usually build with `colcon build` inside the workspace (depending on how the repo is structured).

Example pattern (adjust if the repo already is a workspace):

```bash
cd ~/ros2_ws
colcon build
source install/setup.bash
```

---

## 5) Launch Commands

### 5.1 RViz Visualization
```bash
ros2 launch dobot_rviz dobot_rviz.launch.py
```
### JOint States GUI
```bash
ros2 run joint_state_publisher_gui joint_state_publisher_gui
```
### 5.2 Robot Bringup
```bash
ros2 launch dobot_bringup_v4 dobot_bringup_ros2.launch.py
```

---

## 6) Robot Mode Note (ROS Control)

To control the robot from ROS:
- Switch the robot to **TCP mode** (instead of **online** mode).

---

## 7) RobotStudio Notes (New Features)

- Change braking mode  
- Resistance  

---

## 8) Runtime Debugging (ROS Introspection)

List what is running:

```bash
ros2 node list
ros2 topic list
ros2 service list
```

---

## 9) Working with Services (Type → Interface → Call)

### 9.1 Get the service type (message type)
Example:

```bash
ros2 service type /dobot_bringup_ros2/srv/EnableRobot
```

### 9.2 Show the interface definition
Example:

```bash
ros2 interface show dobot_msgs_v4/srv/EnableRobot
```

### 9.3 Service call template
```bash
ros2 service call <service_name> <service_type> "{<field_name>: <value>}"
```

### 9.4 Call examples

Enable robot:
```bash
ros2 service call /dobot_bringup_ros2/srv/EnableRobot dobot_msgs_v4/srv/EnableRobot "{}"
```

Set speed factor:
```bash
ros2 service call /dobot_bringup_ros2/srv/SpeedFactor dobot_msgs_v4/srv/SpeedFactor "{ratio: 10}"
```

Move joint command (example payload exactly as provided):
```bash
ros2 service call /dobot_bringup_ros2/srv/MovJ dobot_msgs_v4/srv/MovJ "{mode: true, a: 0.0, b: 0.0, c: 0.0, d: 0.0, e: 0.0, f: 0.0, param_value: []}"
```

---

## 10) Common Robot Actions (Checklist)

- Start/Stop Drag  
- Enable SafeSkin  
- Speed Factor  
- Clear Error  

(These are often exposed as services/topics depending on the package setup.)

---

## Python Client


```python

from dobot_msgs_v4.srv import DisableRobot

import rclpy
from rclpy.node import Node


class MinimalClientAsync(Node):

    def __init__(self):
        super().__init__('minimal_client_async')
        self.cli = self.create_client(DisableRobot, '/dobot_bringup_ros2/srv/DisableRobot')
        while not self.cli.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('service not available, waiting again...')
        self.req = DisableRobot.Request()

    def send_request(self):  # (self, a, b):
        # self.req.a = a
        # self.req.b = b
        self.future = self.cli.call_async(self.req)
        rclpy.spin_until_future_complete(self, self.future)
        return self.future.result()


def main(args=None):
    rclpy.init(args=args)

    minimal_client = MinimalClientAsync()
    response = minimal_client.send_request()


    minimal_client.destroy_node()
    rclpy.shutdown()


if __name__ == '__main__':
    main()


```    




## 11) Exercise 1 — Pick & Place (ROS)

**Goal:** Do a pick-and-place exercise in ROS based on the provided demo.

Suggested steps:
1. Launch bringup + RViz.
2. Identify services for:
   - enabling robot
   - setting speed
   - moving joints / moving linear
   - end-effector control (gripper / suction)
3. Execute a sequence:
   - Move above pick point  
   - Move down to pick point  
   - Close gripper / enable suction  
   - Lift up  
   - Move above place point  
   - Move down  
   - Open gripper / disable suction  
   - Return home  
4. Write down (or script) the exact ROS service calls used.

**Extra:** Control the end-effector as part of the sequence.

---

## 12) MoveIt (Important ROS Distro Note)

You wrote:

```bash
sudo apt install ros-humble-moveit
sudo apt install ros-humble-rmw-cyclonedds-cpp
```

⚠️ **Note:** Those are **Humble** packages, but the rest of this tutorial is **humble**.

If your system is humble, you generally want the matching humble packages (names typically start with `ros-humble-...`) instead of `ros-humble-...`.

(If you *intentionally* run Humble in a separate environment/PC, keep them separate and do not mix installs in the same ROS setup.)
