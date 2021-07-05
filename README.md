# ros2_sandbox
\\-------------------------------------\

Tutorial on running ROS2 via docker \
https://docs.ros.org/en/galactic/Guides/Run-2-nodes-in-single-or-separate-docker-containers.html?highlight=docker

docker hub link \
https://hub.docker.com/r/osrf/ros2/

NVIDIA ACCELERATED ROS2 docker image \
https://developer.nvidia.com/blog/accelerating-ai-modules-for-ros-and-ros-2-on-jetson/

interesting thing ... \
https://docs.ros.org/en/foxy/Tutorials/Building-Realtime-rt_preempt-kernel-for-ROS-2.html

\\--------------------------------------\
## commands
```
docker pull osrf/ros:foxy-desktop
docker run -it osrf/ros:foxy-desktop
```

by default, the ros2 foxy docker image uses fastrtp dds, 
```
sudo apt install ros-galactic-rmw-fastrtps-cpp
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
