# 📘 Install RealSense SDK + ORB-SLAM3 Dense on Jetson Orin NX (JetPack 6.2)

This guide installs:

- Intel RealSense SDK  
- OpenCV 4.10 (with CUDA)  
- Pangolin  
- Eigen3  
- ORB-SLAM3 Dense for ROS2  

Tested on **Jetson Orin NX — JetPack 6.2**.

---

## 1️⃣ Install RealSense SDK on Jetson Orin NX

```bash
cd ~
git clone https://github.com/jetsonhacks/jetson-orin-librealsense.git
cd jetson-orin-librealsense
```

### Verify tar package

```bash
sha256sum -c install-modules.tar.gz.sha256
```

### Extract & install modules

```bash
tar -xzf install-modules.tar.gz
cd install-modules
sudo ./install-realsense-modules.sh
```

Successful output:

```
All kernel modules installed and loaded successfully
```

### Reboot

```bash
sudo reboot
```

---

## 2️⃣ Install RealSense apt repository

Unplug the RealSense camera.

### Register the RealSense public key

```bash
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE \
|| sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-key F6E65AC044F831AC80A06380C8B3A55A6F3EFCDE
```

### Add repo

```bash
sudo add-apt-repository "deb https://librealsense.intel.com/Debian/apt-repo $(lsb_release -cs) main" -u
```

### Install SDK

```bash
sudo apt-get install librealsense2-utils
sudo apt-get install librealsense2-dev
```

### Test

```bash
realsense-viewer
```

---

## 3️⃣ Create ORB-SLAM3 Workspace

```bash
mkdir ~/orbslam3_ws
cd ~/orbslam3_ws
mkdir src
cd src
git clone https://github.com/ros-perception/vision_opencv
cd ..
colcon build
```

Copy `orbslam3_dense_ros2` into `src/`.

---

# 4️⃣ Dependencies for orbslam3_dense_ros2

## Eigen3

```bash
sudo apt install libeigen3-dev
```

## Pangolin

```bash
cd ~
git clone https://github.com/stevenlovegrove/Pangolin
cd Pangolin
git checkout v0.8
mkdir build && cd build
cmake ..
make -j$(nproc)
sudo make install
```

---

# 5️⃣ Install OpenCV 4.10 (CUDA Accelerated)

## Update & dependencies

```bash
sudo apt update
sudo apt upgrade
```

```bash
sudo apt install build-essential cmake pkg-config unzip yasm git checkinstall \
libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev \
libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libxvidcore-dev libx264-dev \
libmp3lame-dev libopus-dev libvorbis-dev ffmpeg libva-dev libdc1394-25 libdc1394-dev \
libxine2-dev libv4l-dev v4l-utils libgtk-3-dev libtbb-dev libatlas-base-dev gfortran \
libprotobuf-dev protobuf-compiler libgoogle-glog-dev libgflags-dev libgphoto2-dev \
libeigen3-dev libhdf5-dev doxygen
```

### v4l soft link

```bash
sudo ln -s /usr/include/libv4l1-videodev.h /usr/include/linux/videodev.h
```

---

## Download OpenCV

```bash
cd ~
mkdir opencv && cd opencv
wget -O opencv.zip https://github.com/opencv/opencv/archive/refs/tags/4.10.0.zip
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/refs/tags/4.10.0.zip
unzip opencv.zip
unzip opencv_contrib.zip
```

---

## Build OpenCV

```bash
cd opencv-4.10.0
mkdir build && cd build

cmake .. -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_TBB=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D WITH_CUDA=ON \
-D BUILD_opencv_cudacodec=OFF \
-D WITH_CUDNN=ON \
-D OPENCV_DNN_CUDA=ON \
-D CUDA_ARCH_BIN=7.5 \
-D WITH_V4L=ON \
-D WITH_QT=OFF \
-D WITH_OPENGL=ON \
-D WITH_GSTREAMER=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D OPENCV_PC_FILE_NAME=opencv.pc \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_PYTHON3_INSTALL_PATH=~/.virtualenvs/opencv/lib/python3.12/site-packages/ \
-D PYTHON_EXECUTABLE=~/.virtualenvs/opencv/bin/python \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv/opencv_contrib-4.10.0/modules \
-D INSTALL_PYTHON_EXAMPLES=OFF \
-D INSTALL_C_EXAMPLES=OFF \
-D BUILD_EXAMPLES=OFF
```

### Compile & install

```bash
make -j$(nproc)
sudo make install
sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'
sudo ldconfig
```

---

# 6️⃣ Modify ORB-SLAM3 Config Paths

Edit:

```
orbslam3_dense_ros2/src/main.cpp
```

Set:

```cpp
std::string voc_file = "/home/jetson/orbslam3_ws/src/orbslam3_dense_ros2/orb_slam3/Vocabulary/ORBvoc.txt.bin";
std::string settings_file = "/home/jetson/orbslam3_ws/src/orbslam3_dense_ros2/orb_slam3/config/RGB-D/RealSense_D435i.yaml";
```

---

# 7️⃣ Build

```bash
colcon build --packages-select orbslam3_dense_ros2 \
  --cmake-args -DCMAKE_BUILD_TYPE=Release
```

---

# ▶️ How to Run

### 1. Start RealSense RGBD

```bash
./run_rgbd.sh
```

### 2. Start ORB-SLAM3 Dense

```bash
ros2 run orbslam3_dense_ros2 orb_slam3_main
```

### 3. RViz2

```bash
rviz2 -d ~/orbslam3_ws/orb.rviz
```

---

# System Monitor

```bash
sudo tegrastats
```


