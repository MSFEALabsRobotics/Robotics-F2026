# Unitree Go2 — ROS 2 Connection and Motion Test

**Target:** Ubuntu 22.04 with ROS 2 Humble already installed.

**Official repository:**  
https://github.com/unitreerobotics/unitree_ros2

This guide installs the message packages on your computer, connects to the robot, and tests basic commands. Nothing is installed on the dog.

> These instructions follow Unitree’s ROS source but have not been tested on this lab’s robot.
> The robot must expose its Ethernet DDS interface and have its sport service running.
> Do not use these Humble installation commands inside a Jazzy environment.

## 1. Install dependencies — once

Use a terminal without the previous Python virtual environment active.

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

For Humble, a separate CycloneDDS source build is not needed for this setup.

## 2. Build the Unitree message packages — once

```bash
cd ~
git clone https://github.com/unitreerobotics/unitree_ros2.git

cd ~/unitree_ros2/cyclonedds_ws

source /opt/ros/humble/setup.bash

colcon build --symlink-install \
    --packages-up-to unitree_go unitree_api
```

If the repository already exists, reuse it instead of cloning again.

Load the packages:

```bash
source ~/unitree_ros2/cyclonedds_ws/install/setup.bash
```

Verify:

```bash
ros2 pkg prefix unitree_go
ros2 pkg prefix unitree_api
```

Both commands should print installation paths.

| Package | Purpose |
|---|---|
| `unitree_go` | Interpret robot-state messages |
| `unitree_api` | Construct motion requests and interpret responses |

You do not rebuild every time you connect.

## 3. Connect Ethernet and set the computer IP

Connect the computer to the Go2 using Ethernet.

Find the interface:

```bash
ip -br link
ip -br -4 addr
```

An interface might be named `enp3s0` or `enx001122334455`.

In Ubuntu, open:

**Settings → Network → Wired connection settings → IPv4 → Manual**

Enter:

| Setting | Value |
|---|---|
| Computer IP | `192.168.123.99` |
| Netmask | `255.255.255.0` |
| Gateway | Leave blank |
| DNS | Leave blank for this connection |

Apply and reconnect the wired connection.

> This is the computer’s IP, not the robot’s IP.
> Use your lab’s assigned address if different. Computers sharing the network need unique addresses.

Verify, replacing `enp3s0` with your actual interface:

```bash
ip -4 addr show dev enp3s0
```

Look for:

```text
192.168.123.99/24
```

## 4. Create a reusable setup file — once

```bash
mkdir -p ~/go2_lab
nano ~/go2_lab/setup_go2_ros2.sh
```

Paste the following. **Change `enp3s0` to your Ethernet interface.**

```bash
source /opt/ros/humble/setup.bash
source "$HOME/unitree_ros2/cyclonedds_ws/install/setup.bash"

export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
export ROS_DOMAIN_ID=0
export ROS_LOCALHOST_ONLY=0

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

## 5. Load the environment and discover topics

Power on the robot and wait for startup.

```bash
source ~/go2_lab/setup_go2_ros2.sh

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

The available topics depend on firmware and active services.

> You can discover topic names without the Unitree message packages.
> Reading or publishing their custom messages requires the packages.
> No additional driver launch is needed for this direct DDS test.

## 6. Read robot state before testing motion

If `/sportmodestate` is listed:

```bash
ros2 topic echo /sportmodestate \
    unitree_go/msg/SportModeState \
    --qos-reliability best_effort \
    --once
```

If only `/lf/sportmodestate` is available:

```bash
ros2 topic echo /lf/sportmodestate \
    unitree_go/msg/SportModeState \
    --qos-reliability best_effort \
    --once
```

**Success:** a state message appears containing fields such as position, velocity, and IMU information.

If neither command receives data, resolve the connection before proceeding.

## 7. Test basic commands

Before sending commands:

- Place the robot on a clear, level floor.
- Keep the remote ready and know its stop procedure.
- Keep slack in the Ethernet cable.
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

> ROS reporting that a message was published does not prove the robot accepted or completed the command.

To inspect responses, open another terminal:

```bash
source ~/go2_lab/setup_go2_ros2.sh

ros2 topic echo /api/sport/response \
    unitree_api/msg/Response \
    --qos-reliability best_effort
```

## 8. Optional: short forward movement

First stand the robot up and confirm it is ready to walk.

This test sends a forward velocity of `0.1 m/s` approximately 20 times over 2 seconds, then attempts to send a stop.

Paste the entire block into a terminal:

```bash
(
    source ~/go2_lab/setup_go2_ros2.sh

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

**Ctrl+C attempts cleanup, but a disconnected cable or failed process can prevent the stop from reaching the robot. The remote remains necessary.**

## 9. Troubleshooting

| Problem | Check |
|---|---|
| No robot topics | Ethernet address, interface name, DDS domain, robot DDS access |
| Package not found | Build succeeded and workspace was sourced |
| Topics appear but no state data | Actual topic name and endpoint QoS |
| Publisher waits for a subscriber | Robot sport service and DDS connectivity |
| Messages publish but no movement | Robot mode, response messages, competing controllers |
| `/cmd_vel` does nothing | This direct interface uses `/api/sport/request`; `/cmd_vel` needs a bridge |
| Old settings remain active | Source the setup file and restart the ROS daemon |

Inspect a topic’s endpoints:

```bash
ros2 topic info /api/sport/request --verbose
```

## 10. Next session

The installation and build are already complete.

Simply connect Ethernet, power on the robot, then run:

```bash
source ~/go2_lab/setup_go2_ros2.sh
ros2 topic list -t
```

Verify state reception before sending motion commands.

## Sources

- [Official Unitree ROS 2 repository](https://github.com/unitreerobotics/unitree_ros2)
- [Motion operation IDs and ROS publisher](https://github.com/unitreerobotics/unitree_ros2/blob/master/example/src/include/common/ros2_sport_client.h)
- [Motion parameter implementation](https://github.com/unitreerobotics/unitree_ros2/blob/master/example/src/src/common/ros2_sport_client.cpp)
