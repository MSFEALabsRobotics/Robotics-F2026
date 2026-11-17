
<img width="1672" height="941" alt="image" src="https://github.com/user-attachments/assets/c574ba11-8f93-4c54-bfb3-67f808d7d49e" />


# Robotics Final Project – Autonomous Snack Bar and Delivery Challenge

## 1. Project Overview

Each team will develop an autonomous robotic system capable of **preparing and delivering snack orders** based on customer requests.

The project integrates the **Dobot CR10 robotic arm**, a **custom 3D-printed servo-actuated gripper**, **computer vision**, **QR-code detection**, **natural-language interaction**, and a **mobile robot dog**.

The objective is to create a complete robotic workflow: understand an order, identify and collect the correct snacks, pack them into a container, load the container onto the robot dog, and deliver it to the requested table.

**No liquid pouring is required.** The challenge uses individually packaged, dry snacks to make manipulation and transport safer.

## 2. Custom Gripper and Camera Mount

Since the original robotic gripper is unavailable, **each team must design and manufacture its own gripper**.

Similar to the camera-support assignment completed earlier in the semester, students must:

- Design a **custom 3D-printed gripper** compatible with the Dobot CR10.
- Use a **servo motor** to actuate the gripper.
- Design a mounting system that accommodates **both the camera and the gripper** on the robotic arm.
- Develop the necessary electronics and software to control gripping and releasing.
- Ensure the gripper can reliably handle **different packaged snack shapes and sizes**, as well as the order container when required.

**AI-assisted CAD design ("vibe CADing") is encouraged**, just as in the camera-support assignment. Teams may use AI tools to generate, modify, and iterate on their CAD designs, but they are responsible for verifying dimensions, printability, mechanical fit, and functionality.

Teams are encouraged to develop creative, lightweight, and practical designs.

## 3. Challenge Environment

- Several types of **individually packaged snacks**—such as chocolate bars, granola bars, biscuits, and small packets of crackers—will be placed **randomly on a table within the robot's working area**.
- Empty **delivery boxes or trays** will be stored in a rack or dispenser, identified using a **QR code**.
- The Dobot CR10 will use its camera to identify and locate the requested snacks and assemble the order inside a delivery container.
- A **mobile robot dog**, equipped with a secure carrying tray, will transport the completed order to designated tables.

## 4. Autonomous Operation

**Example customer request:**

> I want 2 chocolate bars and 1 granola bar for Table 4.

The robotic system must automatically perform the following sequence:

1. **Order interpretation:** Understand the requested snack types, quantities, and destination table.
2. **Snack detection:** Identify and locate the correct products among randomly positioned snacks using computer vision.
3. **Container retrieval:** Locate the box or tray dispenser using its QR code and retrieve an empty delivery container.
4. **Snack manipulation:** Grasp the requested snacks using the team's custom gripper.
5. **Order assembly:** Place the snacks into the delivery container, repeating until all requested quantities are collected.
6. **Order verification:** Check that the selected snacks and quantities match the customer request.
7. **Robot loading:** Place the completed container securely on the robot dog's carrying tray.
8. **Autonomous delivery:** Command the robot dog to navigate to the requested table.
9. **Task completion:** Confirm that the order has been delivered.

## 5. Technical Requirements

Each team is responsible for integrating the following components:

- **Mechanical design:** Custom servo-actuated gripper, 3D-printed camera/gripper mount, and reliable handling of packaged products.
- **Computer vision:** Detection and localization of snacks placed in different positions and orientations.
- **QR-code recognition:** Identification and localization of the delivery-container dispenser.
- **Robot manipulation:** Motion planning, grasping, container retrieval, and accurate object placement.
- **User interaction:** Processing natural-language customer orders.
- **Order management:** Tracking snack types and quantities to complete the requested order.
- **System integration:** Communication between the robotic arm, control computer, and mobile robot dog.
- **Mobile robotics:** Autonomous navigation and delivery to the requested table.

## 6. Final Challenge

On demonstration day, teams will receive **unseen customer orders** involving different snack types, quantities, and destination tables.

The locations and orientations of the snack packages will be **randomized**. Teams must detect and adapt to these changes rather than rely entirely on predefined pick positions.

The system should complete each order **autonomously**, without dropping products, mixing up items, or losing the delivery container during robot-dog navigation.

**The winning system will be the one that completes orders most reliably, accurately, and autonomously, with minimal human intervention.**

The project combines mechanical design, 3D printing, embedded systems, perception, robotic manipulation, natural-language interaction, and autonomous navigation into one integrated application.
