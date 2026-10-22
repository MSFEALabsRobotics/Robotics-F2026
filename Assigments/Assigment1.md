<img width="1055" height="1491" alt="image" src="https://github.com/user-attachments/assets/50ce672e-0dd7-45e3-ae33-6329d13375b6" />



# Robotics Assignment: QR Code Tracking Using ROS 2 and CR10 Robot

## Objective

Develop a ROS 2 application that allows the **CR10 robot to automatically follow a selected QR code** using a camera mounted on the robot.

---

## Tasks

### 1. Camera and QR Code Detection

- Connect a camera to ROS 2 and publish its images.
- Implement a QR code detector of your choice (OpenCV, YOLO, or another library).
- Detect QR codes and extract their encoded strings and image positions.

### 2. Camera Mount Design

- Design and fabricate a camera holder that can be attached to the CR10 robot.
- The holder must be secured using **4 bolts**.
- Position the camera **off-center**, leaving enough space for a gripper to be installed later.
- Ensure the camera remains firmly attached during robot movement.

### 3. Robot Control Using ServoJ

- Control the CR10 robot through ROS 2.
- Use the **ServoJ** function instead of the previously used **MovJ** function, since ServoJ is more suitable for continuous tracking.
- Implement a control loop that moves the robot to follow the selected QR code.
- Use appropriate speed limits and safety precautions.

### 4. QR Code Selection Through ROS 2

- Create a ROS 2 topic that receives a string specifying which QR code the robot should track.
- The robot must **follow only the QR code whose encoded text matches the published string**.
- When the published string changes, the robot must automatically switch to tracking the newly selected QR code.
- If the selected QR code is not detected, the robot must stop tracking until it becomes visible.

#### Example

Publish the first target:

```bash
ros2 topic pub --once /target_qr std_msgs/msg/String "{data: 'QR1'}"
```

The robot follows `QR1`.

Change the target:

```bash
ros2 topic pub --once /target_qr std_msgs/msg/String "{data: 'QR3'}"
```

The robot automatically switches to following `QR3`.

---

## Expected Result

A working ROS 2 application in which the CR10 robot continuously follows a selected QR code using its mounted camera, with the ability to switch targets dynamically by publishing a new string.

---

## Deliverables

1. **ROS 2 Package:** Detection and robot-control nodes.
2. **Camera Holder:** CAD model and fabricated mounting part.
3. **Demonstration Video:** Show QR code tracking and switching between at least two QR codes.
4. **README:** Brief instructions explaining how to launch and test the system.

---

## Safety

- Test the detection and control logic without robot motion first.
- Use conservative motion limits.
- Maintain a safe workspace.
- Stop robot motion if the selected QR code is lost.
