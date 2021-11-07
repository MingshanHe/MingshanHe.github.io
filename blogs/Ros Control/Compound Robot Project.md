# Compound Robot Project



## Collaborate Robot Configuration



### 1. Environment Setup

#### 1.1 PC Operating System

|           Operating System           | Network Card |
| :----------------------------------: | :----------: |
| Linux with PREEMPT_RT patched kernel |  100BASE-TX  |

#### 1.2 Install Dependencies

```bash
sudo apt-get install build-essential bc curl ca-certificates fakeroot gnupg2 libssl-dev lsb-release libelf-dev bison flex -y
```

#### 1.3 Prepare for the Real-Time Kernal

  The kernal of the current PC can be known by the order `uname -r`, and find out the nearest version to this kernal from the website: [https://www.kernel.org/pub/linux/kernel/projects/rt/](https://www.kernel.org/pub/linux/kernel/projects/rt/) and down load it by following orders.

```bash
sudo curl -SLO https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.14.250.tar.xz 
sudo curl -SLO https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.14.250.tar.sign
sudo curl -SLO https://www.kernel.org/pub/linux/kernel/projects/rt/4.14/patch-4.14.250-rt124.patch.xz
sudo curl -SLO https://www.kernel.org/pub/linux/kernel/projects/rt/4.14/patch-4.14.250-rt124.patch.sign
```

  After download, unzip them as order.

```bash
sudo xz -d linux-4.14.250.tar.xz
sudo xz -d patch-4.14.250-rt124.patch.xz
```

  inspect the completeness of the `sign` file

```bash
sudo gpg2 --verify linux-4.14.250.tar.sign
```

  Please Remember the Key ID Number, like `6092693E` and run the same order for the `patches` file. But it's not important for the kernal compile.

#### 1.4 Compile the Real-Time Kernal

Unzip :

```bash
sudo tar xf linux-4.14.250.tar
cd linux-4.14.250
sudo patch -p1 < ../patch-4.14.250-rt124.patch
```

Config the kernal:

```bash
sudo make oldconfig
```

And there are the **choices** [1-5] which is need to chose **5** and continue as **Enter**. 

Start Compile:

```bash
sudo fakeroot make -j8 deb-pkg
```

Install the package:

```bash
sudo dpkg -i ../linux-headers-4.14.250-rt124
```

Until the last package, it will be stopped. And you can `ctrl+c` to skip and complete it without any error. Then, you can install the package:

```bash
sudo dpkg -i ../linux-headers-4.14.250-rt124*.deb ../linux-image-4.14.250-rt124_*.deb
```

**Notice: Don't install the last package which is not compile successfully**

#### 1.5 Choose the kernal

  Please reboot your computer and chose the kernal by entering the advanced options.

#### 1.6 RCI-Client Third-Party Library

* boost
* googletest
* eigen
* oroccos-kdl

#### 1.7 Intel-Realsense D435 Camera ROS Wrapper

  Install the ROS distribution and there are **two** sources to install real-sense camera from the **ROS distribution** and **Real-Sense distribution**. And I recommond the ros distribution which is simpler than the other one.

##### ROS distribution

```bash
sudo apt-get install ros-$ROS_DISTRO-realsense2-camera
sudo apt-get install librealsense2-dev
```

  And the later method can be found at the github website: [https://github.com/IntelRealSense/realsense-ros](https://github.com/IntelRealSense/realsense-ros) and [https://github.com/IntelRealsense/librealsense](https://github.com/IntelRealsense/librealsense).

