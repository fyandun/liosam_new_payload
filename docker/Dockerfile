#Download base image ubuntu 16.04
FROM ros:melodic-ros-base

# Suggestion taken from here: https://stackoverflow.com/questions/20635472/using-the-run-instruction-in-a-dockerfile-with-source-does-not-work
# Uses bash instead of sh
SHELL ["/bin/bash", "-c"]

# LABEL about the custom image
LABEL maintainer="davidrus@andrew.cmu.edu"
LABEL version="0.1"
LABEL description="This is custom Docker Image for \
running FrankenSam SLAM system."

# Disable Prompt During Packages Installation
ARG DEBIAN_FRONTEND=noninteractive

# Update Ubuntu Software repository
RUN apt update
RUN apt install curl -y
RUN sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list' && \
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add - && \
apt update

RUN apt install ros-melodic-desktop-full -y
RUN apt install ros-melodic-navigation \
ros-melodic-robot-localization \
ros-melodic-robot-state-publisher \
ros-melodic-velodyne \
ros-melodic-tf \
ros-melodic-cv-bridge \
ros-melodic-pcl-conversions \
ros-melodic-pcl-ros \
ros-melodic-image-transport \
ros-melodic-velodyne-pointcloud -y

RUN apt install git vim wget unzip cmake -y
RUN apt install libgoogle-glog-dev libgflags-dev libatlas-base-dev libeigen3-dev libsuitesparse-dev -y

RUN apt install build-essential  pkg-config libgtk-3-dev \
libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
libxvidcore-dev libx264-dev libjpeg-dev libpng-dev libtiff-dev \
gfortran openexr libatlas-base-dev python3-dev python3-numpy \
libtbb2 libtbb-dev libdc1394-22-dev qtbase5-dev -y

# Install Ceres
RUN mkdir ~/install/CeresSolver -p && \
cd ~/install/CeresSolver && \
wget https://github.com/ceres-solver/ceres-solver/archive/refs/tags/1.14.0.tar.gz && \
tar zxf 1.14.0.tar.gz && \
mkdir ceres-bin && \
cd ceres-bin && \
cmake ../ceres-solver-1.14.0 && \ 
make -j8 && \
make -j8 test && \
sudo make -j8 install

RUN wget -O ~/install/gtsam.zip https://github.com/borglab/gtsam/archive/4.0.2.zip && \
cd ~/install/ && \
unzip gtsam.zip -d ~/install/ && \
cd ~/install/gtsam-4.0.2/ && \
mkdir build && cd build && \
cmake -DGTSAM_BUILD_WITH_MARCH_NATIVE=OFF -DCMAKE_BUILD_TYPE=Release .. && \
sudo make install -j8 && \
cd ~/install/ && \
rm gtsam.zip

RUN mkdir ~/install/opencv_3.3.1 -p && \
cd ~/install/opencv_3.3.1 && \ 
git clone https://github.com/opencv/opencv.git && \
cd opencv && \
git checkout 3.3.1 -b v3.3.1 && \
cd .. && \
git clone https://github.com/opencv/opencv_contrib.git && \
cd opencv_contrib && \
git checkout 3.3.1 -b v3.3.1 && \
cd .. && \
mkdir opencv_build && \
cd opencv_build && \
mkdir ../opencv_install

RUN cd ~/install/opencv_3.3.1/opencv_build && \
cmake -D WITH_TBB=ON \
    -D WITH_V4L=ON \
    -D WITH_OPENMP=ON \
    -D WITH_IPP=ON \
    -D CMAKE_BUILD_TYPE=RELEASE \
    -D BUILD_EXAMPLES=OFF \
    -D WITH_NVCUVID=ON \
    -D WITH_CUDA=ON \
    -D BUILD_DOCS=OFF \
    -D BUILD_PERF_TESTS=OFF \
    -D BUILD_TESTS=OFF \
    -D WITH_CSTRIPES=ON \
    -D WITH_OPENCL=ON \
    -D WITH_QT=ON \
    -D WITH_OPENGL=ON \
    -D OPENCV_PYTHON3_INSTALL_PATH=$cwd/OpenCV-py3/lib/python3.5/site-packages \
    -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \
    -D CMAKE_INSTALL_PREFIX=../opencv_install ../opencv

RUN cd ~/install/opencv_3.3.1/opencv_build && make 
RUN cd ~/install/opencv_3.3.1/opencv_build && make install

RUN apt install tmux -y

RUN mkdir ~/catkin_ws/src -p && \
cd ~/catkin_ws/src && \
git clone https://github.com/fyandun/liosam_new_payload.git 

RUN export CMAKE_MODULE_PATH=$CMAKE_MODULE_PATH:/root/install/opencv_3.3.1/opencv_install
RUN echo $CMAKE_MODULE_PATH

RUN cd ~/catkin_ws && \
source /opt/ros/melodic/setup.bash && \
catkin_make -j 8 --cmake-args -DGTSAM_INCLUDE_DIRS=/usr/local/include/ 

