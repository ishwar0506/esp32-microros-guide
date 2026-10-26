# micro-ROS ESP32-S3 Setup — Complete Guide

A complete, copy-paste ready setup guide for configuring micro-ROS on any new ESP32-S3 device from scratch.

## Prerequisites
- **Ubuntu 22.04** (ROS2 Humble)
- **ESP32-S3 board**
- **Arduino IDE 2.x** installed
- **Docker** installed

---

## PART 1 — ROS2 & micro-ROS Agent Setup

### Install ROS2 Humble (skip if already installed)
```bash
sudo apt update && sudo apt install -y curl gnupg lsb-release
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update && sudo apt install -y ros-humble-desktop python3-rosdep python3-colcon-common-extensions
```

### Source ROS2
```bash
source /opt/ros/humble/setup.bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "export ROS_DOMAIN_ID=0" >> ~/.bashrc
source ~/.bashrc
```

### Create micro-ROS workspace and build agent
```bash
mkdir -p ~/microros_ws/src && cd ~/microros_ws
git clone -b humble https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup
sudo apt update && sudo rosdep init && rosdep update
rosdep install --from-paths src --ignore-src -y
sudo apt install -y python3-pip
colcon build
source install/local_setup.bash
```

### Build micro-ROS agent
```bash
ros2 run micro_ros_setup create_agent_ws.sh
ros2 run micro_ros_setup build_agent.sh
source install/local_setup.bash
```

---

## PART 2 — Arduino Library Setup

### Clone micro_ros_arduino library (Humble branch)
```bash
cd ~/Arduino/libraries
git clone -b humble https://github.com/micro-ROS/micro_ros_arduino.git
```

### Copy ESP32 board files for ESP32-S3
```bash
cd ~/Arduino/libraries/micro_ros_arduino/src
cp -r esp32 esp32s3
```

### Verify serial transport
```bash
grep -r "set_microros_transports" ~/Arduino/libraries/micro_ros_arduino/src/micro_ros_arduino.h
```

---

## PART 3 — Arduino IDE Board Settings

In Arduino IDE, configure these under the **Tools** menu:

| Setting | Value |
| :--- | :--- |
| **Board** | ESP32S3 Dev Module |
| **USB Mode** | Hardware CDC and JTAG |
| **Upload Speed** | 921600 |
| **Flash Size** | 4MB or 8MB |
| **Partition Scheme** | Huge APP (3MB No OTA) |
| **Port** | /dev/ttyACM0 |

---

## PART 4 — Arduino Sketch

See the `esp32_test_node/esp32_test_node.ino` file in this repository for the code.

---

## PART 5 — Flash & Test (Every Time)

### Terminal 1 — Start Agent
```bash
source ~/.bashrc
cd ~/microros_ws && source install/local_setup.bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0 -b 115200
```
*Then press the **RESET** button on the ESP32.*

### Terminal 2 — Verify
```bash
source ~/.bashrc
cd ~/microros_ws && source install/local_setup.bash

ros2 topic list                  # Should show /esp32_counter
ros2 topic echo /esp32_counter   # Should show incrementing data
ros2 node list                   # Should show /esp32_test_node
```

---

## PART 6 — Troubleshooting Cheatsheet

**Find correct port:**
```bash
ls /dev/tty* | grep -E "USB|ACM"
```

**Port permission denied:**
```bash
sudo chmod 666 /dev/ttyACM0
# or permanent fix:
sudo usermod -a -G dialout $USER
```

**Port busy (Arduino IDE holding it):**
```bash
sudo fuser -k /dev/ttyACM0
```

**Topics not showing:**
```bash
ROS_DOMAIN_ID=0 ros2 topic list
```

**Docker files permission error:**
```bash
sudo rm -rf ~/Arduino/libraries/micro_ros_arduino
```

**Verbose agent logging:**
```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyACM0 -b 115200 -v6
```
