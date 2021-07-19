# ROS2 rviz visualization + Standalone Fast DDS Publisher
Date: June 19, 2021
### Claim: successfully verified the following approach,
- Standalone Fast-DDS Publisher and
- ROS2 rviz visualization 

Below are steps to reproduce.
## 1 Create ROS2(Foxy) docker image with libGL support (rviz2)
Two approaches to create ros2 with libGL support are verified working, and either way works.
### 1.a Build from customized Dockerfile 
https://github.com/NVIDIA/nvidia-docker/issues/534#issuecomment-436054364

Following the link above, I customized the Dockerfile and build the image successfully which is verified to support rviz2 ! \
The Dockefile and start up scripts have been included in this repo.

### 1.b Use OSRF official tool - rocker
Another approach, which I also verified working, is to use rocker. But compared to pure docker file, its not as clean as the former.  \
https://github.com/osrf/rocker

After 1.a or 1.b, enable x11 support (if not done before and only need enable once)
```
xhost +local:docker
```

create network (if not done before and only need create once)
```
docker network create foo
```

load the docker image using 
```
docker run --rm -it --gpus all  \
        --net foo \
        --name master \
        -e DISPLAY \
        -e TERM   \
        -e QT_X11_NO_MITSHM=1   \
        -e XAUTHORITY=/tmp/.docker4go9n3_z.xauth \
        -v /tmp/.docker4go9n3_z.xauth:/tmp/.docker4go9n3_z.xauth   \
        -v /tmp/.X11-unix:/tmp/.X11-unix   \
        -v /etc/localtime:/etc/localtime:ro \
        -v /home/xiaojun/sandbox/libs:/opt/libs \
        -v /home/xiaojun/sandbox/data:/opt/data \
        ros2:nvidia
```
Details are included in the startup_script file. Once docker image runs, verified rviz2 is working inside ROS2 docker container
```
rviz2
```

## 2. FastDDS docker image setup
Download fast-dds docker image (.tar) from link below. Tested v2.0.0 and v2.3.0 and both version work. \
https://eprosima.com/index.php/downloads-all

and follow the install instructions to load and run docker image\
https://github.com/eProsima/Fast-DDS
```
sudo apt-get install docker.io
docker load -i ubuntu-fast-dds:v2.0.0.tar
docker run -it \
        --privileged \
        --net foo \
        -e DISPLAY=$DISPLAY \
        -v /tmp/.X11-unix:/tmp/.X11-unix \
        -v /home/xiaojun/sandbox/data:/opt/data \
        ubuntu-fast-dds:v2.0.0
```

## 3. Program and build FAST-DDS Publisher 
As an example, I chose the ROS2 LaserScan topic to publish by Fast-DDS standalone publisher, and visualize the message using RVIZ2. The major reason to chose "LaserScan" message is because it contains .header variabel which further includes timestamp value.

### 3.1 prepare IDL files
copy 3 .idl files from ROS2 container 
```
cp /opt/ros/foxy/share/sensor_msgs/msg/LaserScan.idl /opt/data/
cp /opt/ros/foxy/share/std_msgs/msg/Header.idl /opt/data/
cp /opt/ros/foxy/share/builtin_interfaces/msg/Time.idl /opt/data/
```

In FAST-DDS container,
```
mkdir wspace_fastdds/LaserScan_test
cd wspace_fastdds/LaserScan_test
mv /opt/data/LaserScan.idl .
mv /opt/data/Header.idl .
mv /opt/data/Time.idl .
```
In LaserScan.idl file, change 
```#include "std_msgs/msg/Header.idl"```
into 
```#include "Header.idl"```

add underscore so that 
```
    struct _LaserScan {
```

In Header.idl file, change 
```#include "builtin_interfaces/msg/Time.idl"```
into 
```#include "Time.idl"```
add underscore so that 
```
    struct _Header {
```

In Time.idl file, add underscore so that 
```
    struct _Time {
```

### 3.2 run fastrtpsgen
```
fastrtpsgen -typeros2 -example CMake LaserScan.idl
```
in the generated source file LaserScanPublisher.cxx, change
```
Wparam.topic.topicName = "LaserScanPubSubTopic";
```
into 
```
Wparam.topic.topicName = "rt/scan";
```

Then add LaserScan data publishing codes, as explained in the source file include in this repo.

### 3.3 build
```
cmake .
make
```

### 3.4 run 
```
./LaserScan publisher
```

## 4. Verify RVIZ2 visualization result
Then inside ROS2 container, run rviz2 and select /Scan topic to visualize, and change Fixed Frame to "laser_frame"
then the laser scan data should be visualized.


if run 
```
ros2 topic list
```
the /scan topic will be listed.


## other useful references
standalone FASTDDS talk to ROS2 \
https://github.com/briancaglioni/FASTDDS_ROS2_Pub_Sub_Guide

fastDDS quick start \
https://github.com/eProsima/Fast-DDS

Tutorial on running ROS2 via docker \
https://docs.ros.org/en/galactic/Guides/Run-2-nodes-in-single-or-separate-docker-containers.html?highlight=docker

docker hub link \
https://hub.docker.com/r/osrf/ros2/

NVIDIA ACCELERATED ROS2 docker image \
https://developer.nvidia.com/blog/accelerating-ai-modules-for-ros-and-ros-2-on-jetson/

interesting thing ... \
https://docs.ros.org/en/foxy/Tutorials/Building-Realtime-rt_preempt-kernel-for-ROS-2.html

## commands to run basic ros2 nodes
```
docker pull osrf/ros:foxy-desktop
docker run -it osrf/ros:foxy-desktop
```
Note: the above configuration does not support rviz as libGL support is not present.

## ross dds options
by default, the ros2 foxy docker image uses fastrtp dds, 
```
sudo apt install ros-foxy-rmw-fastrtps-cpp
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```

but can be switched to cyclonedds by
```
sudo apt install ros-foxy-rmw-cyclonedds-cpp
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
```
as described here, \
https://docs.ros.org/en/galactic/Installation/DDS-Implementations/Working-with-Eclipse-CycloneDDS.html

or rti dds(if commercial or trial license installed).

I tried one node (publisher) using cyclonedds, the other node (subscriber) using fastrtp, and the two nodes can communicate without issue!
