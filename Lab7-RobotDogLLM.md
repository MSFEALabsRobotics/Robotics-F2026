# Robotics Lab — Unitree Go2 Connection and Test

Use the prepared ROS 2 Humble workspace with CycloneDDS and the Unitree packages installed. This procedure uses **WSL NAT mode**.

## 1. Set a static IP address on Windows

Connect the computer to the robot network. Set the IPv4 address of the Windows network adapter used for this connection to the static address assigned in the lab, on the `192.168.123.x` subnet. Do not use an address already assigned to another device.

The scripts below use:

- Robot: `192.168.123.161`
- Jetson: `192.168.123.18`

**Windows static IP settings: photo to be added.**

<!-- Add the Windows static IP configuration screenshot here. -->

## 2. Start the relay in Windows PowerShell

Start WSL, then open **Windows PowerShell as Administrator** and paste the complete script below. Leave this PowerShell window open throughout the lab. Stop any previous relay before starting a new one.

```powershell
& {
    $ErrorActionPreference = "Stop"
    $distro = ""   # Empty = default WSL distribution
    $sources = @("192.168.123.161", "192.168.123.18")
    $ports = 7410..7451

    # Detect addresses.
    $wslArgs = @()
    if ($distro) { $wslArgs = @("-d", $distro) }
    $routeText = & wsl.exe @wslArgs -- ip -j -4 route get $sources[0]
    if ($LASTEXITCODE -ne 0) { throw "Cannot read the WSL route." }
    $route = @($routeText | ConvertFrom-Json)[0]
    $wslIP = $route.prefsrc
    $gatewayIP = $route.gateway
    $windowsIP = (Find-NetRoute -RemoteIPAddress $sources[0] |
        Where-Object { $_.IPAddress } |
        Select-Object -First 1).IPAddress

    if (!$wslIP -or !$gatewayIP -or !$windowsIP) {
        throw "Address detection failed. This script requires WSL NAT mode."
    }

    Write-Host "Windows=$windowsIP  Gateway=$gatewayIP  WSL=$wslIP"

    $busy = @(Get-NetUDPEndpoint -ErrorAction SilentlyContinue |
        Where-Object { $_.LocalPort -in $ports })
    if ($busy.Count) {
        $busy | Format-Table LocalAddress,LocalPort,OwningProcess
        throw "Stop the previous relay before starting this one."
    }

    $rule = "WSL-DDS-Relay-7410-7451"
    Get-NetFirewallRule -Name $rule -ErrorAction SilentlyContinue |
        Remove-NetFirewallRule
    New-NetFirewallRule -Name $rule -DisplayName $rule `
        -Direction Inbound -Action Allow -Protocol UDP `
        -LocalAddress $windowsIP -LocalPort "7410-7451" `
        -RemoteAddress $sources -Profile Any | Out-Null

    function Endpoint($address, $port) {
        [System.Net.IPEndPoint]::new(
            [System.Net.IPAddress]::Parse($address), [int]$port)
    }

    $rx = @{}; $dest = @{}; $seen = @{}; $tx = $null
    try {
        $tx = [System.Net.Sockets.UdpClient]::new((Endpoint $gatewayIP 0))
        foreach ($port in $ports) {
            $rx[$port] = [System.Net.Sockets.UdpClient]::new(
                (Endpoint $windowsIP $port))
            $dest[$port] = Endpoint $wslIP $port
        }

        Write-Host "Relay running on UDP 7410-7451. Leave open; Ctrl+C stops it."
        while ($true) {
            foreach ($port in $ports) {
                for ($n = 0; $n -lt 32 -and $rx[$port].Available -gt 0; $n++) {
                    $sender = Endpoint "0.0.0.0" 0
                    $data = $rx[$port].Receive([ref]$sender)
                    $address = $sender.Address.ToString()
                    if ($address -in $sources) {
                        [void]$tx.Send($data, $data.Length, $dest[$port])
                        $key = "${address}:$port"
                        if (!$seen[$key]) {
                            Write-Host "Forwarding $address -> WSL UDP $port"
                            $seen[$key] = $true
                        }
                    }
                }
            }
            Start-Sleep -Milliseconds 5
        }
    }
    finally {
        foreach ($socket in $rx.Values) { $socket.Dispose() }
        if ($null -ne $tx) { $tx.Dispose() }
    }
}
```

## 3. Configure ROS 2 in the WSL terminal

Use a WSL terminal with ROS 2 and the Unitree workspace sourced.

**Before running:** fill `robot_ports = []` and `jetson_ports = []` with the latest discovery results for the robot and Jetson. Enter comma-separated integer port numbers inside each pair of brackets. Both lists must be filled; otherwise, the script stops.

Paste the complete block below into the WSL terminal:

```bash
export ROS_DOMAIN_ID=0
export ROS_LOCALHOST_ONLY=0
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

if xml="$(python3 - <<'PY'
import ipaddress
import json
import subprocess
import sys
from xml.sax.saxutils import quoteattr

# Paste the latest discovery results here.
robot_ports = []
jetson_ports = []

if not robot_ports or not jetson_ports:
    raise SystemExit("Fill robot_ports and jetson_ports first.")

robot, jetson = "192.168.123.161", "192.168.123.18"
route = json.loads(subprocess.check_output(
    ["ip", "-j", "-4", "route", "get", robot], text=True
))[0]
local = str(ipaddress.IPv4Address(route["prefsrc"]))
interface = quoteattr(route["dev"])

command = (
    "$ErrorActionPreference='Stop'; "
    "(Find-NetRoute -RemoteIPAddress '192.168.123.161' | "
    "Where-Object { $_.IPAddress } | "
    "Select-Object -First 1).IPAddress"
)
windows = str(ipaddress.IPv4Address(subprocess.check_output(
    ["powershell.exe", "-NoProfile", "-NonInteractive",
     "-Command", command], text=True
).strip()))

peers = f'<Peer Address="{local}"/>'
for address, ports in [(robot, robot_ports), (jetson, jetson_ports)]:
    for port in sorted(set(ports)):
        if type(port) is not int or not 1 <= port <= 65535:
            raise SystemExit(f"Invalid port: {port}")
        peers += f'<Peer Address="{address}:{port}"/>'

print(f"Detected: Windows={windows}, WSL={local}, "
      f"interface={route['dev']}", file=sys.stderr)

print(f"""<CycloneDDS><Domain Id="0">
  <General>
    <Interfaces><NetworkInterface name={interface}/></Interfaces>
    <AllowMulticast>false</AllowMulticast>
    <ExternalNetworkAddress>{windows}</ExternalNetworkAddress>
    <ExternalNetworkMask>255.255.255.255</ExternalNetworkMask>
  </General>
  <Discovery>
    <ParticipantIndex>auto</ParticipantIndex>
    <MaxAutoParticipantIndex>20</MaxAutoParticipantIndex>
    <Peers>{peers}</Peers>
  </Discovery>
  <Compatibility><ManySocketsMode>false</ManySocketsMode></Compatibility>
</Domain></CycloneDDS>""")
PY
)"; then
    export CYCLONEDDS_URI="$xml"
    ros2 topic list --no-daemon --spin-time 20
else
    echo "Configuration failed; previous DDS setting preserved."
fi
```

Wait for the topic list and confirm that `/api/sport/request` appears before continuing. Keep this WSL terminal open for the next step; its exported settings apply to this terminal.

## 4. Test the robot dog

Place the robot in a clear area, ready for movement, with its remote stop control available.

Run the following command in the **same WSL terminal**:

```bash
ros2 topic pub -r 20 /api/sport/request unitree_api/msg/Request '{header: {identity: {api_id: 1008}}, parameter: "{\"x\": 0.3, \"y\": 0.0, \"z\": 0.0}"}'
```

Press **Ctrl+C** to stop publishing. Use the robot's stop control to ensure it has stopped before approaching it.

After completing the test, press **Ctrl+C** in the PowerShell window to stop the relay.
