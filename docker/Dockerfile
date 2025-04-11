FROM osrf/ros:noetic-desktop-full

RUN apt-get update


RUN mkdir -p /root/ros_ws/src

RUN apt-get update && apt-get install -y git python3-pip mlocate
RUN apt-get install -y curl
RUN apt-get install -y ros-noetic-effort-controllers
RUN apt-get install -y ros-noetic-moveit
RUN apt-get install -y ros-noetic-aruco-detect
RUN apt-get install -y ros-noetic-rosbash
RUN apt-get install -y ros-noetic-ros-comm
RUN apt-get install -y ros-noetic-catkin
RUN apt-get install -y ros-$ROS_DISTRO-realsense2-camera
RUN apt-get install -y ros-$ROS_DISTRO-realsense2-description


SHELL ["/bin/bash", "-c"]
RUN mkdir -p /ros_ws/src

WORKDIR /ros_ws/src
RUN source /opt/ros/noetic/setup.bash
RUN wstool init .
RUN wstool merge https://gist.github.com/alekulyn/558c06074775189792cc7658a7bbe143/raw/712819dcb8d0b4e258d1be6681221d75c05cfda1/baxter_package.rosinstall 
RUN wstool update

WORKDIR /ros_ws/src/dope
RUN python3 -m pip install -r requirements.txt

WORKDIR /ros_ws/src/camera_info_manager_py
RUN curl https://gist.githubusercontent.com/alekulyn/f9b747b0f43a29c715bfbd48886b1743/raw/6f005578ec7328a845d964c41fb14b159b78d196/setup.py -o setup.py

WORKDIR /ros_ws
RUN source /opt/ros/noetic/setup.bash\
  && catkin_make -DCATKIN_BLACKLIST_PACKAGES="dope"\
  && source devel/setup.bash

WORKDIR /ros_ws
RUN source /ros_ws/devel/setup.bash\
  && rosdep install --from-paths /ros_ws/src/camera_info_manager_py --rosdistro noetic -y\
  && catkin_make\
  && echo 'source /ros_ws/devel/setup.bash' >> ~/.bashrc\
  && source ~/.bashrc
RUN source /ros_ws/devel/setup.bash\
  && rosdep install --from-paths /ros_ws/src/dope -i --ignore-src --rosdistro noetic -y\
  && catkin_make\
  && echo 'source /ros_ws/devel/setup.bash' >> ~/.bashrc\
  && source ~/.bashrc

# This is a super bad hack to get around the fact there is an issue with the generated ActionFeedback message.
# This seems to be an issue of a type mismatch after upgrading to python3
COPY resources/_FollowJointTrajectoryActionFeedback.py /opt/ros/noetic/lib/python3/dist-packages/control_msgs/msg/_FollowJointTrajectoryActionFeedback.py
WORKDIR /ros_ws
