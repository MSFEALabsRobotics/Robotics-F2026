# USB Webcam in WSL 2 with ROS 2 Humble

Use a Logitech C270 webcam in WSL 2, publish its images with `usb_cam`, and view them with `rqt_image_view`.

**Working settings:** 640 × 480, MJPEG, 15 FPS.

## 1. Install USB support on Windows (once)

Open **Windows PowerShell as Administrator**:

```powershell
winget install --interactive --exact dorssel.usbipd-win
```

Close and reopen PowerShell as Administrator after installation.

## 2. Attach the webcam to WSL

Keep a WSL terminal open and close Windows applications using the webcam.

In **PowerShell**, list the connected USB devices:

```powershell
usbipd list
```

Find the Logitech webcam's BUSID. In this setup it was `2-2`; replace this value in the commands below if yours differs.

Share the device **once**, in PowerShell as Administrator:

```powershell
usbipd bind --busid 2-2
```

Attach it to WSL:

```powershell
usbipd attach --wsl --busid 2-2
```

> While attached to WSL, the webcam is unavailable to Windows applications.

## 3. Verify the webcam in WSL

Run in **WSL**:

```bash
lsusb
ls -l /dev/video*
```

The Logitech webcam should appear in `lsusb`. In this setup, `/dev/video0` and `/dev/video1` appeared; `/dev/video0` is the capture device used below.

If `lsusb` is unavailable, install it:

```bash
sudo apt update
sudo apt install usbutils
```

## 4. Give your user camera access (once)

Run in **WSL**:

```bash
sudo usermod -aG video "$USER"
newgrp video
```

`newgrp video` activates the group in the current terminal. Continue with the commands below in that terminal. Existing terminals may need a new login session or their own `newgrp video` command.

## 5. Install the ROS 2 camera packages (once)

These commands assume ROS 2 Humble is already installed.

```bash
sudo apt update
sudo apt install ros-humble-usb-cam ros-humble-rqt-image-view
```

## 6. Start the camera

In **WSL terminal 1**:

```bash
source /opt/ros/humble/setup.bash

ros2 run usb_cam usb_cam_node_exe --ros-args \
  -p video_device:=/dev/video0 \
  -p pixel_format:=mjpeg2rgb \
  -p image_width:=640 \
  -p image_height:=480 \
  -p framerate:=15.0
```

Keep this terminal running. Press **Ctrl+C** to stop the camera node.

> In the tested setup, the default YUYV mode at 30 FPS timed out. The MJPEG configuration above worked successfully.

## 7. View the camera image

In **WSL terminal 2**:

```bash
source /opt/ros/humble/setup.bash
ros2 run rqt_image_view rqt_image_view
```

Select **`/image_raw`** from the topic dropdown.

To check the image topic's received frame rate, run in another sourced WSL terminal:

```bash
ros2 topic hz /image_raw
```

## 8. Return the webcam to Windows

Stop the camera node with **Ctrl+C**, then run in **PowerShell**:

```powershell
usbipd detach --busid 2-2
```

## Next session

After restarting WSL or unplugging the webcam:

1. Open WSL.
2. Run `usbipd list` in PowerShell to check the current BUSID.
3. Run `usbipd attach --wsl --busid 2-2`, using the correct BUSID.
4. Start the camera using step 6.
5. Open the viewer using step 7.

The installation and `bind` commands do not normally need to be repeated.

## Troubleshooting

| Symptom | Action |
| --- | --- |
| No `/dev/video*` devices | Check USB attachment with `usbipd list` in Windows and `lsusb` in WSL. If the webcam appears in `lsusb` only, investigate WSL camera-driver support. |
| Cannot open device / permissions error | Run step 4, then check `id` and `ls -l /dev/video*`. Your user should belong to `video`. |
| Missing `default_cam.yaml` calibration file | This does not prevent basic video streaming. Calibration is needed for tasks requiring accurate camera geometry. |
| `Select timeout, exiting...` | Use the MJPEG settings in step 6. If the timeout persists, inspect `sudo dmesg \| tail -n 30`. |

## References

- [Microsoft: Connect USB devices to WSL](https://learn.microsoft.com/en-us/windows/wsl/connect-usb)
- [usbipd-win: WSL support](https://github.com/dorssel/usbipd-win/wiki/WSL-support)
- [ROS usb_cam package](https://github.com/ros-drivers/usb_cam)
