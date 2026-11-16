# Robotics Food Challenge

**An autonomous food-preparation and delivery challenge using the Dobot CR10 and Unitree Go2**

## 1. Project Overview

The **Robotics Food Challenge** asks student teams to build an intelligent robotic food-service system that can adapt to ingredients it has **never seen during development**. The system combines a **Dobot CR10 robotic arm** for food handling and preparation with a **Unitree Go2 quadruped robot** for final delivery.

The central idea is simple: **the tools are known, but the ingredients are a surprise.** Teams can design and test their tool rack, manipulation strategies, perception pipeline, and control software in advance. However, the actual ingredients are revealed only at challenge time. Instead of replaying a fixed sequence, the robot must recognize the available ingredients, ask the customer what they want, select suitable tools, prepare the order, and deliver it to the correct table.

This project brings together perception, human–robot interaction, task planning, robot manipulation, and mobile robot navigation in one end-to-end scenario.

## 2. Challenge Scenario

Imagine a small automated food-service station. A customer approaches and the system receives a collection of surprise ingredients. The robot must:

1. **Inspect the workspace** using a camera or other available sensors.
2. **Identify the ingredients** and estimate their locations and relevant properties.
3. **Interact with the customer** to ask what they would like prepared from those ingredients and obtain a table number.
4. **Form a preparation plan**, selecting only feasible steps and the required tools.
5. **Pick up and use tools** from a predefined rack, such as a fork, knife, or spatula.
6. **Handle and assemble the food** while minimizing spills, drops, and unnecessary contact.
7. **Place the finished order on a serving tray** at the designated handoff location.
8. **Coordinate with the Unitree Go2**, placing or handing off the tray only when the robot dog is safely positioned.
9. **Deliver the order** to the customer's designated table using autonomous navigation.

The goal is **adaptability**, not a memorized recipe. If the customer requests something the system cannot safely prepare, the robot should explain its limitation and suggest an achievable alternative.

## 3. Available Hardware and Environment

### Dobot CR10 robotic arm

The CR10 is responsible for the manipulation tasks: detecting object locations, picking up tools, moving ingredients, assembling food, and positioning the completed tray. Teams may use the existing gripper, approved tool adapters, cameras, and their own software interfaces.

### Tool station

A common set of **known tools** is available before the challenge. Examples include:

- Fork
- Spatula
- Knife (only if explicitly approved and supervised)
- Serving tray and ingredient containers

Teams may calibrate tool pickup locations and develop routines for each tool ahead of time. **Tools are predictable; ingredient identity, quantity, and arrangement are not.**

### Unitree Go2 quadruped

The Go2 acts as the delivery robot. It must navigate from the preparation station to the assigned table after receiving confirmation that the tray is ready and secured. The CR10 and Go2 must coordinate their handoff so neither robot moves unexpectedly during loading.

## 4. Technical Objectives

### A. Computer Vision and Perception

Students should develop a perception system capable of identifying surprise ingredients and estimating their positions relative to the robot. Potential techniques include object detection, image segmentation, depth sensing, and camera-to-robot calibration.

The perception system should also determine whether an object is graspable and whether additional observations are needed. Uncertain detections should be handled explicitly rather than treated as reliable facts.

### B. AI-Based Decision-Making

The system should convert the detected ingredient list and the customer's request into a realistic sequence of actions. An LLM or another planning approach may be used for **high-level reasoning**, for example:

- What can be prepared from the available ingredients?
- Which tools are appropriate?
- In what order should the steps be performed?
- What should the robot do if an item cannot be grasped or a step fails?

High-level AI decisions should be translated into **validated robot actions**. An AI-generated instruction must not directly bypass motion, workspace, or safety constraints.

### C. Robot Manipulation and Motion Planning

Teams will implement tool pickup and return, grasp planning, collision-aware trajectories, ingredient transfer, and food assembly. The system should account for different object sizes, shapes, and placements.

ROS 2 and MoveIt may be used to organize motion planning and robot control. The robot should verify successful grasps and placements whenever feedback is available.

### D. Human–Robot Interaction

The customer interface may be text-based, voice-based, or graphical. At a minimum, it should:

1. Present or confirm the detected ingredient list.
2. Ask the customer what they want prepared.
3. Confirm that the requested order is feasible.
4. Ask for and confirm the table number.
5. Report when the order is ready or if assistance is needed.

### E. Multi-Robot Coordination and Navigation

The CR10 and Go2 form one system, even though they perform different tasks. Teams must design a clear handoff protocol, including states such as `WAITING_FOR_TRAY`, `READY_TO_LOAD`, `TRAY_SECURED`, `DELIVERING`, and `DELIVERED`.

The Go2 must reach the requested table through the approved demonstration area while avoiding obstacles. A simple known-table map or more advanced autonomous navigation may be used, depending on the available lab setup.

## 5. Rules of the Challenge

1. **Surprise ingredients:** The actual ingredients are announced or provided only at evaluation time.
2. **Known tool set:** The tools and their storage locations are made available in advance.
3. **Customer-driven task:** The robot must obtain the customer's preferred order and table number through an interface.
4. **Autonomous execution:** After the challenge starts, the main sequence should proceed without manual teleoperation. Clearly identified safety interventions are always allowed.
5. **Flexible planning:** The robot must select tools and adjust its sequence to the provided ingredients rather than execute only a fixed demonstration.
6. **Safe handling:** The system must respect workspace limits, collision constraints, and food-handling procedures.
7. **Graceful failure:** If a task is impossible or unsafe, the robot should stop that step safely and explain the problem rather than improvising dangerously.
8. **Complete delivery:** Successful completion requires both preparation and delivery to the correct table.

> **Important:** Use simulated food, pre-cut ingredients, or other approved low-risk materials for early testing. Any sharp cutting, heating, or direct food consumption requires separate instructor approval, supervision, and appropriate hygiene safeguards. All robot motions must be tested at reduced speed before full demonstrations.

## 6. Suggested System Architecture

A possible software architecture is:

```text
Camera / Sensors
      |
      v
Ingredient Recognition + Pose Estimation
      |
      v
Customer Interface ---> Order + Table Number
      |                       |
      +-----------+-----------+
                  |
                  v
        High-Level Task Planner
                  |
                  v
     Action Validation / Safety Checks
                  |
                  v
        CR10 Manipulation Controller
                  |
                  v
        Tray Ready / Handoff Signal
                  |
                  v
        Go2 Delivery Controller
                  |
                  v
          Destination Table
```

Teams are free to propose different architectures as long as the modules are integrated and the system remains safe and testable.

## 7. Suggested Development Milestones

| Milestone | Demonstration |
| --- | --- |
| **1. Station setup** | Calibrated work area, tool rack, robot interfaces, and safety boundaries |
| **2. Perception** | Detect and localize a changing set of ingredient objects |
| **3. Tool manipulation** | Pick up, use, and return at least one tool reliably |
| **4. Customer interaction** | Receive an order and table number and confirm feasibility |
| **5. Autonomous preparation** | Execute a planned sequence with more than one manipulation step |
| **6. Robot handoff** | Safely transfer the tray to the Go2 at a designated location |
| **7. Delivery** | Navigate to a selected table and report completion |
| **8. Final integration** | Run the complete system with surprise ingredients |

## 8. Expected Team Deliverables

- **Source code:** Well-organized repositories for perception, planning, manipulation, and delivery modules.
- **Setup documentation:** Installation steps, dependencies, hardware configuration, calibration, and launch commands.
- **System diagram:** Major software components, data flow, and communication between robots.
- **Demonstration:** A live end-to-end attempt using ingredients selected at challenge time.
- **Short technical report:** Design decisions, test results, limitations, failures, and improvements.
- **Safety checklist:** Robot workspace limits, emergency stop procedure, permitted tools, and handoff conditions.

## 9. Suggested Evaluation Criteria

The following is a **proposed rubric**; the instructor may adjust the weights before the challenge.

| Criterion | Weight |
| --- | ---: |
| Recognition of unknown ingredients and their locations | 20% |
| Autonomous tool selection and manipulation | 25% |
| Appropriate task planning and customer interaction | 20% |
| CR10–Go2 coordination and correct table delivery | 20% |
| Robustness, safety, and recovery from errors | 15% |
| **Total** | **100%** |

Teams should receive credit for a robust, explainable system even if a surprise ingredient or unforeseen condition prevents the entire sequence from finishing. Unsafe behavior should not be rewarded merely because a task completes.

## 10. Final Goal

The **Robotics Food Challenge** is not simply about making a robotic arm pick up food. It asks students to build an integrated autonomous system that **perceives an unfamiliar situation, reasons about what is possible, selects tools, performs physical actions, interacts with a human, and collaborates with another robot**.

A successful team will demonstrate that its system can do more than repeat a programmed routine: **it can adapt to a new food-preparation request and safely complete the process from ingredient recognition to table delivery.**
