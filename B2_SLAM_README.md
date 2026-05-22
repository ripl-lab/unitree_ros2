# B2_SLAM_README.md

## Overview

This document outlines the setup process used to configure a development laptop for working with the Unitree B2 SLAM SDK and related repositories.

Tested on:

- Ubuntu 22.04
- ROS2 Humble
- x86_64 architecture

---

# 1. Workspace Structure

The ROS workspace and Unitree SDK repositories are kept separate.

## ROS Workspace

```text
~/pace_ws/src/
    b2_interfaces/
    b2_description/
    unitree_ros2/
```

Only ROS packages (i.e., repositories containing a `package.xml`) should be placed inside the ROS workspace.

## Unitree SDK / SLAM Repositories

```text
~/unitree/
    unitree_sdk2/
    unitree_slam/
```

These repositories are standard C++ projects and are **not** ROS packages.

---

# 2. ROS2 Environment Setup

Install ROS2 Humble for Ubuntu 22.04 following the official ROS2 installation guide.

Verify installation:

```bash
source /opt/ros/humble/setup.bash
echo $ROS_DISTRO
```

Expected output:

```text
humble
```

---

# 3. Build ROS Workspace

From the workspace root:

```bash
cd ~/pace_ws

source /opt/ros/humble/setup.bash

rosdep install --from-paths src --ignore-src -r -y

colcon build --symlink-install
```

---

# 4. Install Unitree SDK2

Clone repository:

```bash
mkdir -p ~/unitree
cd ~/unitree

git clone https://github.com/unitreerobotics/unitree_sdk2.git
```

## Install Dependencies

```bash
sudo apt update

sudo apt install -y \
    cmake \
    g++ \
    build-essential \
    libyaml-cpp-dev \
    libeigen3-dev \
    libboost-all-dev \
    libspdlog-dev \
    libfmt-dev
```

## Build and Install SDK2

```bash
cd ~/unitree/unitree_sdk2

mkdir -p build
cd build

cmake .. -DCMAKE_INSTALL_PREFIX=/opt/unitree_robotics

make -j$(nproc)

sudo make install
```

## Add SDK2 To CMake Prefix Path

```bash
echo 'export CMAKE_PREFIX_PATH=/opt/unitree_robotics:$CMAKE_PREFIX_PATH' >> ~/.bashrc

source ~/.bashrc
```

Verify:

```bash
echo $CMAKE_PREFIX_PATH
```

Expected output should include:

```text
/opt/unitree_robotics
```

---

# 5. Build Unitree SLAM

Download the test routine package from the B2 SDK Development Guide:

:contentReference[oaicite:0]{index=0}

Extract the folder and move `unitree_slam` into:

```text
~/unitree/
```

## Modify CMakeLists.txt

Inside:

```text
~/unitree/unitree_slam/CMakeLists.txt
```

add:

```cmake
execute_process(
    COMMAND uname -m
    OUTPUT_VARIABLE ARCH
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

include_directories(
    ${CMAKE_SOURCE_DIR}/unitree_robotics/include
)

link_directories(
    ${CMAKE_SOURCE_DIR}/unitree_robotics/lib/${ARCH}
)
```

This allows the project to find:
- the required `ros2_idl` headers
- the precompiled Unitree libraries

without manually passing include/linker flags during every build.

## Build

```bash
cd ~/unitree/unitree_slam

mkdir -p build
cd build

cmake .. -DCMAKE_PREFIX_PATH=/opt/unitree_robotics

make -j$(nproc)
```

---

# 6. Verify Build Outputs

To verify successful SLAM compilation:

```bash
find ~/unitree/unitree_slam/build -maxdepth 1 -type f -executable
```

Expected executables include:

```text
demo_b2
demo_mid360
demo_xt16
start_nav
start_mappping
start_relocation
single_nav
recover_nav
```

---

# 7. Notes

- `unitree_slam` is not a ROS package.
- The repository contains precompiled architecture-specific libraries.
- The provided `install.sh` script was not used in this setup.
- SDK2 was installed into:

```text
/opt/unitree_robotics
```

instead of copying libraries into `/usr/local`.

---