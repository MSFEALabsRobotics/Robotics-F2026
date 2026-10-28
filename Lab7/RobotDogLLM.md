# Robotics Session: Connect to the Unitree Robot with ROS 2

## Session objective

Set up a ROS 2 Humble workspace in WSL, configure Cyclone DDS communication through a Windows UDP relay, discover the robot's topics, and run a forward-motion test at **0.3 m/s**.

This guide organizes the supplied lab configuration. The commands have not been tested on the robot as part of preparing this document.

## 1. Lab setup and network

Use the following terminals throughout the session:

| Terminal | Purpose |
| --- | --- |
| WSL / Ubuntu | Install dependencies, build the workspace, configure DDS, and run ROS 2 commands |
| Windows PowerShell **as Administrator** | Run the UDP relay and configure its firewall rule |
| Jetson terminal — robot administrators only | Discover the robot and Jetson DDS ports |

Connect the Windows computer to the robot network before discovery and testing:

| Setting | Value |
| --- | --- |
| Wi-Fi SSID | `RLIL` |
| Wi-Fi password | `unitree123` |
| Windows IP on the robot network | `192.168.123.87` |
| Windows gateway IP toward WSL | `172.28.176.1` |
| WSL IP | `172.28.185.55` |
| Robot IP | `192.168.123.161` |
| Jetson IP | `192.168.123.18` |
| WSL network interface | `eth0` |
| ROS domain ID | `0` |
| Windows relay UDP ports | `7410–7451` |

These addresses are specific to the supplied lab setup. Confirm the current addresses and update every matching value in the scripts if they differ. The Jetson script below discovers **ports for the two configured IP addresses**; it does not discover unknown device IP addresses.

## 2. Install dependencies — WSL

This session assumes ROS 2 Humble and `colcon` are already installed.

```bash
sudo apt update
sudo apt install ros-humble-rmw-cyclonedds-cpp
sudo apt install ros-humble-rosidl-generator-dds-idl
sudo apt install libyaml-cpp-dev
```

## 3. Add the packages — WSL

Clone the repositories inside the workspace's `src` directory. Run these clone commands once; if a repository already exists, use the existing checkout.

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone https://github.com/unitreerobotics/unitree_ros2
git clone https://github.com/ros2/rmw_cyclonedds -b humble
git clone https://github.com/eclipse-cyclonedds/cyclonedds -b releases/0.10.x
```

## 4. Build the workspace — WSL

Source ROS 2 Humble before building. After a successful build, source the workspace overlay.

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
colcon build
source ~/ros2_ws/install/setup.bash
```

## 5. Discover DDS ports — Jetson, robot administrators only

With the robot and Jetson running, execute the following in the **Jetson terminal**. It listens for discovery traffic for **60 seconds**, then prints the two Python lists required in the next step.

```bash
python3 - <<'PY'
import socket
import struct
import time

ROBOT = "192.168.123.161"
JETSON = "192.168.123.18"
found = {ROBOT: set(), JETSON: set()}

def parameters(data, pos, end, endian):
    while pos + 4 <= end:
        pid, size = struct.unpack_from(endian + "HH", data, pos)
        pos += 4

        if pid == 1:
            return

        if pos + size > end:
            return

        yield pid, data[pos:pos + size]
        pos += (size + 3) & ~3

def discovery_locators(data):
    if len(data) < 20 or data[:4] != b"RTPS":
        return

    pos = 20

    while pos + 4 <= len(data):
        kind, flags = data[pos:pos + 2]
        endian = "<" if flags & 1 else ">"
        size = struct.unpack_from(endian + "H", data, pos + 2)[0]

        body = pos + 4
        end = body + size if size else len(data)

        if end > len(data):
            return

        if (kind == 0x15 and flags & 4 and body + 20 <= end
                and data[body + 8:body + 12] == b"\x00\x01\x00\xc2"):

            offset = struct.unpack_from(
                endian + "H", data, body + 2
            )[0]
            payload = body + 4 + offset

            if flags & 2:
                while payload + 4 <= end:
                    pid, length = struct.unpack_from(
                        endian + "HH", data, payload
                    )
                    payload += 4

                    if pid == 1:
                        break

                    payload += (length + 3) & ~3
                else:
                    payload = end

            if payload + 4 <= end:
                encoding = data[payload:payload + 2]

                if encoding in (b"\x00\x02", b"\x00\x03"):
                    pe = "<" if encoding == b"\x00\x03" else ">"

                    for pid, value in parameters(
                            data, payload + 4, end, pe):

                        if pid == 0x32 and len(value) >= 24:
                            locator_kind, port = struct.unpack_from(
                                pe + "iI", value
                            )

                            if locator_kind == 1 and 0 < port < 65536:
                                address = socket.inet_ntoa(value[20:24])
                                yield address, port

        pos = end

s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind(("", 7400))
s.setsockopt(
    socket.IPPROTO_IP,
    socket.IP_ADD_MEMBERSHIP,
    socket.inet_aton("239.255.0.1") + socket.inet_aton(JETSON)
)
s.settimeout(1)

print("Listening for DDS discovery for 60 seconds...", flush=True)
deadline = time.monotonic() + 60

try:
    while time.monotonic() < deadline:
        try:
            data, source = s.recvfrom(65535)
        except socket.timeout:
            continue

        if source[0] not in found:
            continue

        for address, port in discovery_locators(data):
            if address in found:
                found[address].add(port)
finally:
    s.close()

print("\nCopy these results:")
print("robot_ports =", sorted(found[ROBOT]))
print("jetson_ports =", sorted(found[JETSON]))

for address, ports in found.items():
    if not ports:
        print(f"WARNING: No discovery ports captured for {address}.")
PY
```

Copy the complete `robot_ports = [...]` and `jetson_ports = [...]` output lines. If either list is empty, resolve discovery with the robot administrator before continuing.

## 6. Configure the ROS environment and DDS — WSL

Open the Bash startup file:

```bash
nano ~/.bashrc
```

Add the following block at the end of the file. If equivalent ROS setup lines already exist, keep one copy in this order: ROS Humble, workspace overlay, then DDS configuration.

**Before saving, replace both empty port lists with the fresh Jetson discovery results from Step 5.**

```bash
source /opt/ros/humble/setup.bash
source ~/ros2_ws/install/setup.bash

export ROS_DOMAIN_ID=0
export ROS_LOCALHOST_ONLY=0
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

if updated_xml="$(python3 - <<'PY'
import xml.etree.ElementTree as ET

# Replace these with the fresh Jetson discovery results.
robot_ports = [[32936, 33678, 35268, 35630, 35930, 36036, 36575, 37415, 37488, 40512, 40762, 42069, 42957, 43129, 43898, 44125, 44520, 45428, 45825, 46768, 54646, 59763, 60950]
jetson_ports = [50600, 58756, 60098, 60906]

if not robot_ports or not jetson_ports:
    raise SystemExit(
        "Port lists are empty. Insert the discovery results first."
    )

root = ET.Element("CycloneDDS")
domain = ET.SubElement(root, "Domain", Id="0")

general = ET.SubElement(domain, "General")
interfaces = ET.SubElement(general, "Interfaces")
ET.SubElement(interfaces, "NetworkInterface", name="eth0")

for name, value in [
    ("AllowMulticast", "false"),
    ("ExternalNetworkAddress", "192.168.123.87"),
    ("ExternalNetworkMask", "255.255.255.255"),
]:
    ET.SubElement(general, name).text = value

discovery = ET.SubElement(domain, "Discovery")
ET.SubElement(discovery, "ParticipantIndex").text = "auto"
ET.SubElement(discovery, "MaxAutoParticipantIndex").text = "20"

peers = ET.SubElement(discovery, "Peers")

# Discover other ROS processes in this WSL instance.
ET.SubElement(peers, "Peer", Address="172.28.185.55")

for address, ports in [
    ("192.168.123.161", robot_ports),
    ("192.168.123.18", jetson_ports),
]:
    for port in sorted(set(ports)):
        ET.SubElement(
            peers, "Peer", Address=f"{address}:{port}"
        )

compatibility = ET.SubElement(domain, "Compatibility")
ET.SubElement(
    compatibility, "ManySocketsMode"
).text = "false"

print(ET.tostring(root, encoding="unicode"))
PY
)"; then
    export CYCLONEDDS_URI="$updated_xml"
    echo "DDS configuration applied."
    ros2 topic list --no-daemon --spin-time 20
else
    echo "DDS configuration NOT updated."
fi
```

Save the file. Start the Windows relay in Step 7 before applying this configuration in Step 8.

The block includes a 20-second topic discovery command, so it also runs whenever a new interactive Bash terminal loads `~/.bashrc`. If configuration generation fails, it does not replace any previously set `CYCLONEDDS_URI`.

## 7. Start the UDP relay — Windows PowerShell

Open **PowerShell as Administrator** on Windows and paste the complete block below. Confirm that `$windowsIP`, `$gatewayIP`, and `$wslIP` match the current setup.

```powershell
& {
    $ErrorActionPreference = "Stop"

    $windowsIP = "192.168.123.87"
    $gatewayIP = "172.28.176.1"
    $wslIP = "172.28.185.55"

    $allowedSources = @(
        "192.168.123.161",
        "192.168.123.18"
    )

    $ports = 7410..7451

    $occupied = @(
        Get-NetUDPEndpoint -ErrorAction SilentlyContinue |
            Where-Object { $_.LocalPort -in $ports }
    )

    if ($occupied.Count -gt 0) {
        $occupied |
            Format-Table LocalAddress,LocalPort,OwningProcess

        $occupied.OwningProcess |
            Sort-Object -Unique |
            ForEach-Object {
                Get-CimInstance Win32_Process -Filter "ProcessId = $_" |
                    Format-List ProcessId,ParentProcessId,Name,CommandLine
            }

        Write-Host "Relay NOT started: ports are occupied."
        Write-Host "Stop the previous relay, or send this output."
        return
    }

    $ruleName = "WSL-DDS-Relay-7410-7451"

    # Replace only this script's firewall rule.
    if (Get-NetFirewallRule -Name $ruleName -ErrorAction SilentlyContinue) {
        Remove-NetFirewallRule -Name $ruleName
    }

    New-NetFirewallRule `
        -Name $ruleName `
        -DisplayName "WSL DDS relay UDP 7410-7451" `
        -Direction Inbound `
        -Action Allow `
        -Protocol UDP `
        -LocalAddress $windowsIP `
        -LocalPort "7410-7451" `
        -RemoteAddress $allowedSources `
        -Profile Any | Out-Null

    $receivers = @{}
    $tx = $null
    $seen = @{}

    try {
        $tx = [System.Net.Sockets.UdpClient]::new(
            [System.Net.IPEndPoint]::new(
                [System.Net.IPAddress]::Parse($gatewayIP), 0
            )
        )

        foreach ($port in $ports) {
            $receivers[$port] = [System.Net.Sockets.UdpClient]::new(
                [System.Net.IPEndPoint]::new(
                    [System.Net.IPAddress]::Parse($windowsIP), $port
                )
            )
        }

        Write-Host "DDS relay running: Windows UDP 7410-7451 -> WSL."
        Write-Host "Leave this window open. Ctrl+C stops the relay."

        while ($true) {
            foreach ($port in $ports) {
                $rx = $receivers[$port]

                for ($n = 0; $n -lt 32 -and $rx.Available -gt 0; $n++) {
                    $sender = [System.Net.IPEndPoint]::new(
                        [System.Net.IPAddress]::Any, 0
                    )

                    $packet = $rx.Receive([ref]$sender)
                    $sourceIP = $sender.Address.ToString()

                    if ($sourceIP -in $allowedSources) {
                        $destination = [System.Net.IPEndPoint]::new(
                            [System.Net.IPAddress]::Parse($wslIP), $port
                        )

                        [void]$tx.Send(
                            $packet, $packet.Length, $destination
                        )

                        $key = "${sourceIP}:$port"

                        if (-not $seen.ContainsKey($key)) {
                            Write-Host "Forwarding $sourceIP -> WSL UDP $port"
                            $seen[$key] = $true
                        }
                    }
                }
            }

            Start-Sleep -Milliseconds 5
        }
    }
    finally {
        foreach ($rx in $receivers.Values) {
            $rx.Dispose()
        }

        if ($null -ne $tx) {
            $tx.Dispose()
        }
    }
}
```

Leave this PowerShell window open throughout testing. `Ctrl+C` stops the relay. If ports are occupied, identify the process shown in the output and stop a previous relay if applicable before retrying.

## 8. Apply the configuration and list topics — WSL

Return to the WSL terminal and reload the Bash configuration:

```bash
source ~/.bashrc
```

The startup block sources ROS Humble and the workspace, applies the DDS configuration, and performs a 20-second topic discovery attempt. Confirm that it prints `DDS configuration applied.`

List the topics:

```bash
ros2 topic list
```

To repeat the discovery command used in the configuration block:

```bash
ros2 topic list --no-daemon --spin-time 20
```

## 9. Test with the robot — WSL

Coordinate the motion test with the robot administrator. Use a clear test area and keep the robot's normal stop control ready. Only one operator should issue motion commands during this test.

### Move forward at 0.3 m/s

Run the supplied command in the WSL terminal:

```bash
ros2 topic pub -r 20 /api/sport/request unitree_api/msg/Request '{header: {identity: {api_id: 1008}}, parameter: "{\"x\": 0.3, \"y\": 0.0, \"z\": 0.0}"}'
```

This continuously publishes the supplied request at **20 Hz**, with `x = 0.3`, `y = 0.0`, and `z = 0.0`.

Press `Ctrl+C` in the publishing terminal to stop sending requests. Stopping the publisher is not a confirmed robot stop; use the administrator's established stopping procedure and verify that the robot has stopped before ending the test.

## Session completion checklist

- [ ] Dependencies installed and repositories available in `~/ros2_ws/src`.
- [ ] Workspace built successfully and sourced.
- [ ] Robot administrator supplied nonempty discovery port lists.
- [ ] Current network addresses and port lists entered in the scripts.
- [ ] Windows relay running.
- [ ] DDS configuration applied and robot topics visible.
- [ ] Forward-motion test completed under administrator supervision.
- [ ] Publisher stopped, robot confirmed stopped, and relay stopped when finished.
