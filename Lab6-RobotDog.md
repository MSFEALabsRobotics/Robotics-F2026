# Unitree Go2 — ROS 2 Test in Your Existing Workspace

Use your existing `~/ros2_ws` workspace to install the Unitree message packages, read robot state, and send basic motion commands.

**Target:** Ubuntu 22.04 with ROS 2 Humble already installed.

**Official repository:**  
https://github.com/unitreerobotics/unitree_ros2

> Your existing workspace must also use Humble for these commands.
> Do not source Humble into a Jazzy workspace.
> The robot must expose Ethernet DDS access and have its sport service running.
> These instructions have not been tested on your robot.

## 1. Install dependencies — once

Open a terminal without the previous Python virtual environment active.

```bash
source /opt/ros/humble/setup.bash

sudo apt update

sudo apt install -y \
    git \
    build-essential \
    python3-colcon-common-extensions \
    ros-humble-rmw-cyclonedds-cpp \
    ros-humble-rosidl-generator-dds-idl \
    libyaml-cpp-dev
```

For this Humble setup, a separate CycloneDDS source build is not needed.

## 2. Add Unitree to your existing workspace — once

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src

git clone https://github.com/unitreerobotics/unitree_ros2.git
```

If Unitree packages are already present inside this workspace, reuse them rather than creating duplicate packages.

Although the repository contains an internal workspace folder, build from your existing workspace root:

```bash
cd ~/ros2_ws

source /opt/ros/humble/setup.bash

colcon build --symlink-install \
    --packages-up-to unitree_go unitree_api
```

Colcon discovers the packages inside the cloned repository.

Load your workspace:

```bash
source ~/ros2_ws/install/setup.bash
```

Verify:

```bash
ros2 pkg prefix unitree_go
ros2 pkg prefix unitree_api
```

Both commands should print paths inside your workspace’s installation.

| Package | Purpose |
|---|---|
| `unitree_go` | Interpret robot-state messages |
| `unitree_api` | Construct motion requests and interpret responses |

You do not rebuild each time you connect.

## 3. Connect Ethernet and configure the computer IP

Connect the computer to the Go2 using Ethernet.

Find the network interface:

```bash
ip -br link
ip -br -4 addr
```

Examples include `enp3s0` and `enx001122334455`.

Open:

**Ubuntu Settings → Network → Wired connection settings → IPv4 → Manual**

Enter:

| Setting | Value |
|---|---|
| Computer IP | `192.168.123.99` |
| Netmask | `255.255.255.0` |
| Gateway | Leave blank |
| DNS | Leave blank for this connection |

Apply and reconnect Ethernet.

> This is the computer’s address, not the robot’s address.
> Use your lab’s assigned settings if different. Computers sharing the network need unique addresses.

Verify, replacing `enp3s0` with your actual interface:

```bash
ip -4 addr show dev enp3s0
```

Look for:

```text
192.168.123.99/24
```

## 4. Create the Go2 environment file — once

Save the setup file in your existing workspace:

```bash
nano ~/ros2_ws/setup_go2_ros2.sh
```

Paste the following.

**Replace `enp3s0` with the Ethernet interface connected to the robot.**

```bash
# Load ROS and the existing workspace.
source /opt/ros/humble/setup.bash
source "$HOME/ros2_ws/install/setup.bash"

# DDS settings for direct Go2 communication.
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DOMAIN_ID=0
export ROS_LOCALHOST_ONLY=0

# Select the Ethernet interface connected to the robot.
export CYCLONEDDS_URI='<CycloneDDS>
  <Domain>
    <General>
      <Interfaces>
        <NetworkInterface name="enp3s0"
                          priority="default"
                          multicast="default"/>
      </Interfaces>
    </General>
  </Domain>
</CycloneDDS>'
```

Save with **Ctrl+O**, press **Enter**, then exit with **Ctrl+X**.

> Source this file only in terminals intended for the Go2.
> Its DDS settings select the robot’s Ethernet interface.

## 5. Discover the robot’s topics

Power on the robot and wait for startup.

```bash
source ~/ros2_ws/setup_go2_ros2.sh

ros2 daemon stop
ros2 daemon start

ros2 topic list -t
```

Look for topics such as:

```text
/sportmodestate
/lf/sportmodestate
/api/sport/request
/api/sport/response
```

Availability depends on firmware and active services.

You can discover topic names without the Unitree message packages. Reading and publishing their custom messages requires the packages.

No additional driver launch is needed for this direct DDS test.

## 6. Read robot state

If `/sportmodestate` is listed:

```bash
ros2 topic echo /sportmodestate \
    unitree_go/msg/SportModeState \
    --qos-reliability best_effort \
    --once
```

If your robot exposes `/lf/sportmodestate` instead:

```bash
ros2 topic echo /lf/sportmodestate \
    unitree_go/msg/SportModeState \
    --qos-reliability best_effort \
    --once
```

**Success:** a state message appears with fields such as position, velocity, and IMU information.

Resolve connection problems before attempting motion.

## 7. Test stop and posture commands

Before sending commands:

- Place the robot on a clear, level floor.
- Keep the remote ready and know its stop procedure.
- Leave slack in the Ethernet cable.
- Stop other motion-control programs.
- Keep the high-level sport service enabled.

Run each command separately.

### Stop

```bash
ros2 topic pub --once \
    /api/sport/request \
    unitree_api/msg/Request \
    '{header: {identity: {api_id: 1003}}, parameter: "{}"}'
```

### Stand up

```bash
ros2 topic pub --once \
    /api/sport/request \
    unitree_api/msg/Request \
    '{header: {identity: {api_id: 1004}}, parameter: "{}"}'
```

Wait until the robot finishes standing and becomes stable.

### Stand down

Run only when the robot is stationary and the area underneath is clear:

```bash
ros2 topic pub --once \
    /api/sport/request \
    unitree_api/msg/Request \
    '{header: {identity: {api_id: 1005}}, parameter: "{}"}'
```

> “Publishing” confirms that ROS sent a message. It does not prove the robot accepted or completed the action.

To inspect responses, open another terminal:

```bash
source ~/ros2_ws/setup_go2_ros2.sh

ros2 topic echo /api/sport/response \
    unitree_api/msg/Response \
    --qos-reliability best_effort
```

## 8. Optional: short forward movement

First stand the robot up and confirm it is ready to walk.

This test sends a forward velocity of `0.1 m/s` approximately 20 times over 2 seconds, then attempts a stop.

Paste the entire block into a terminal:

```bash
(
    source ~/ros2_ws/setup_go2_ros2.sh

    read -r -p "Robot standing, area clear, remote ready? Type GO: " answer
    if [ "$answer" != "GO" ]; then
        exit 0
    fi

    stop_robot() {
        echo "Sending stop request..."

        if ! timeout --signal=INT 5s ros2 topic pub --once \
            /api/sport/request \
            unitree_api/msg/Request \
            '{header: {identity: {api_id: 1003}}, parameter: "{}"}'
        then
            echo "Stop publication failed or timed out. Use the remote stop."
        fi
    }

    trap stop_robot EXIT
    trap 'exit 130' INT
    trap 'exit 143' TERM

    timeout --signal=INT 6s ros2 topic pub \
        --rate 10 \
        --times 20 \
        /api/sport/request \
        unitree_api/msg/Request \
        '{header: {identity: {api_id: 1008}}, parameter: "{\"x\":0.1,\"y\":0.0,\"z\":0.0}"}'
)
```

The outer timeout also limits waiting for DDS discovery.

| Parameter | Meaning |
|---|---|
| `x` | Forward/backward velocity, m/s |
| `y` | Sideways velocity, m/s |
| `z` | Turning rate, rad/s |

This controls velocity, not exact travel distance.

**A disconnected cable or failed process can prevent the stop from reaching the robot. Keep the remote ready.**

## 9. Troubleshooting

| Problem | Check |
|---|---|
| No robot topics | Ethernet address, interface name, DDS domain, robot DDS access |
| Package not found | Build succeeded and `~/ros2_ws/install/setup.bash` was sourced |
| Duplicate package error | Keep only one copy of each Unitree package inside the workspace |
| Topics appear but no state data | Actual topic name and endpoint QoS |
| Publisher waits for a subscriber | Robot sport service and DDS connectivity |
| Messages publish but no movement | Robot mode, response messages, competing controllers |
| `/cmd_vel` does nothing | This interface uses `/api/sport/request`; `/cmd_vel` needs a bridge |
| Old discovery settings remain active | Source the setup file and restart the ROS daemon |

Inspect motion-topic endpoints:

```bash
ros2 topic info /api/sport/request --verbose
```

## 10. Next session

Connect Ethernet, power on the robot, and run:

```bash
source ~/ros2_ws/setup_go2_ros2.sh
ros2 topic list -t
```

Verify state reception before sending motion commands.

**No rebuild is needed unless package sources or build dependencies change.**

## References

- [Official Unitree ROS 2 repository](https://github.com/unitreerobotics/unitree_ros2)
- [Motion IDs and ROS publisher](https://github.com/unitreerobotics/unitree_ros2/blob/master/example/src/include/common/ros2_sport_client.h)
- [Motion parameter implementation](https://github.com/unitreerobotics/unitree_ros2/blob/master/example/src/src/common/ros2_sport_client.cpp)
