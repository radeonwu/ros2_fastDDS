## Purpose
- to test ros2 bag (record/replay) feature with fastDDS 

## Steps
### 1. install ROS2 docker
install osrf/ros:humble-desktop-full by
```docker pull osrf/ros:humble-desktop ```

refer to 
```
https://hub.docker.com/r/osrf/ros/tags 
```
or
```
https://docs.ros.org/en/humble/How-To-Guides/Run-2-nodes-in-single-or-separate-docker-containers.html
```
### 2. install fast DDS docker
download docker image of ubuntu-eprosima-dds-suite:v1.4.0 (as of Jan 2023) from the following download link,
```
https://www.eprosima.com/index.php/component/ars/repository/eprosima-dds-suite
```
then load the docker image and run it,
```
docker load -i ubuntu-eprosima-dds-suite\ v1.4.0.tar
docker run -it         
  --privileged         
  --net foo         
  -e DISPLAY=$DISPLAY         
  -v /tmp/.X11-unix:/tmp/.X11-unix         
  -v /home/xiaojun/sandbox/data:/opt/data         
  ubuntu-eprosima-dds-suite:v1.4.0
```

### revise fastdds publisher code



### run ros2 bag + fastdds publisher
record
```
ros2 bag record /scan
```
and replay
```
ros2 bag play rosbag2_2023_01_11-00_52_28/
```

