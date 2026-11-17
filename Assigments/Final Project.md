
<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/c574ba11-8f93-4c54-bfb3-67f808d7d49e" />


# Robotics Final Project – Autonomous Drink Preparation and Delivery Challenge

## 1. Project Overview

Each team will develop an autonomous robotic system capable of preparing and delivering drinks based on customer requests.

The project integrates the **Dobot CR10 robotic arm**, a **custom 3D-printed servo-actuated gripper**, **computer vision**, **QR-code detection**, and a **mobile robot dog**.

The objective is to create a complete robotic workflow, from understanding a customer's order to preparing the drinks and delivering them to the requested table.

## 2. Custom Gripper and Camera Mount

Since the original robotic gripper is unavailable, **each team must design and manufacture its own gripper**.

Similar to the camera-support assignment completed earlier in the semester, students must:

- Design a custom 3D-printed gripper compatible with the Dobot CR10.
- Use a servo motor to actuate the gripper.
- Design a mounting system that accommodates the camera and gripper on the robotic arm.
- Develop the necessary electronics and software to control gripping and releasing.
- Ensure that the gripper can reliably handle bottles and cups.

**AI-assisted CAD design ("vibe CADing") is encouraged**, just as in the camera-support assignment. Teams may use AI tools to help generate, modify, and iterate on their CAD designs, but must verify dimensions, printability, mechanical fit, and functionality themselves.

Teams are encouraged to develop creative, lightweight, and practical designs.

## 3. Challenge Environment

Several drink bottles, such as orange juice, apple juice, and water, will be placed **randomly within the robot's working area**.

Empty cups will be stored in a vertical cup dispenser or rack, whose location can be identified using a QR code.

A mobile robot dog will serve as the delivery platform, transporting prepared drinks to designated tables.

## 4. Autonomous Operation

**Example customer request:**

> I want 2 cups of orange juice for Table 4.

The robotic system must automatically perform the following sequence:

1. **Order interpretation:** Understand the requested drink, quantity, and destination table.
2. **Bottle detection:** Identify and locate the correct drink bottle among randomly positioned bottles using computer vision.
3. **Cup retrieval:** Locate the vertical cup dispenser using its QR code and retrieve an empty cup.
4. **Bottle manipulation:** Grasp the appropriate bottle using the custom gripper.
5. **Drink preparation:** Pour the requested drink into the cup while minimizing spills.
6. **Repeat:** Prepare the requested number of cups.
7. **Robot loading:** Place the filled cups securely onto the mobile robot dog's carrying tray.
8. **Autonomous delivery:** Command the robot dog to navigate to the requested table.
9. **Task completion:** Confirm that the order has been delivered.

## 5. Technical Requirements

Each team is responsible for integrating the following components:

- **Mechanical design:** Custom servo-actuated gripper and 3D-printed mounting components.
- **Computer vision:** Detection and localization of bottles and cups.
- **QR-code recognition:** Identification and localization of the cup dispenser.
- **Robot manipulation:** Motion planning, gripping, cup retrieval, and controlled pouring.
- **User interaction:** Processing natural-language drink orders.
- **System integration:** Communication between the robotic arm, computer, and mobile robot dog.
- **Mobile robotics:** Autonomous navigation and delivery to the requested table.

## 6. Final Challenge

On the demonstration day, teams will receive customer orders with different drink types, quantities, and table destinations.

Bottle positions will be randomized, requiring the robots to detect and adapt to the environment rather than rely entirely on predefined positions.

**The winning system will be the one that completes orders reliably, accurately, and autonomously, with minimal human intervention.**

The project challenges students to combine mechanical design, embedded systems, perception, robotic manipulation, and autonomous navigation into one complete application.
