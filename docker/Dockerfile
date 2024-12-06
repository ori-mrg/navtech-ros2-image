# Base image for ROS 2 Jazzy
FROM docker.io/ros:foxy AS base

# Use bash as the default shell
SHELL ["/bin/bash", "-c"]

# Set environment variable for non-interactive installations
ENV DEBIAN_FRONTEND=noninteractive

# Set up keyboard configuration
RUN echo "keyboard-configuration keyboard-configuration/layout select us" | debconf-set-selections && \
    echo "keyboard-configuration keyboard-configuration/variant select USA" | debconf-set-selections

# Update and install prerequisites for ROS 2 and IASDK
RUN apt-get update \
 && apt-get install -y \
    build-essential \
    clang \
    g++ \
    protobuf-compiler \
    libprotobuf-dev \
    cmake \
    libssl-dev \
    libffi-dev \
    python3-dev \
    python3 \
    python3-pip \
    software-properties-common \
    curl \
    gnupg \
    lsb-release \
 && rm -rf /var/lib/apt/lists/*

# Add ROS 2 package source
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
    -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | \
    tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Install ROS 2 Foxy
RUN apt-get update && \
    apt-get install -y ros-foxy-desktop && \
    rm -rf /var/lib/apt/lists/*

# Source ROS 2 setup.bash for the current shell
RUN echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc

# Install OpenCV dependencies
RUN apt-get update \
 && apt-get install -y \
    pkg-config \
    libpng-dev \
    libtiff-dev \
    gfortran \
    openexr \
    libgtk-3-dev \
    libavcodec-dev \
    libgstreamer-plugins-base1.0-dev \
    libgstreamer1.0-dev \
    libavformat-dev \
    libswscale-dev \
    libv4l-dev \
    libxvidcore-dev \
    libx264-dev \
    libjpeg-dev \
    libatlas-base-dev \
    libdc1394-dev \
    libopenexr-dev \
 && rm -rf /var/lib/apt/lists/*

# Clone OpenCV repositories
RUN git clone --depth 20 https://github.com/opencv/opencv.git ~/opencv && \
    git clone --depth 20 https://github.com/opencv/opencv_contrib.git ~/opencv_contrib

# Build and install OpenCV
RUN cd ~/opencv && mkdir build && cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
          -D CMAKE_INSTALL_PREFIX=/usr/local \
          -D WITH_CUDA=OFF \
          -D INSTALL_PYTHON_EXAMPLES=ON \
          -D OPENCV_GENERATE_PKGCONFIG=ON \
          -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
          -D OPENCV_ENABLE_NONFREE=ON \
          -D BUILD_EXAMPLES=ON .. && \
    make -j$(nproc) && \
    make install && \
    ldconfig

# Verify OpenCV installation
RUN python3 -c "import cv2; print(cv2.__version__)" && \
    pkg-config --modversion opencv4

# Install Navtech Radar IASDK
# RUN git clone https://bitbucket.org/navtechradar/iasdk-public.git ~/iasdk-public

# Install Colcon (ROS 2 build tool)
RUN apt-get update \
 && apt-get install -y \
    python3-colcon-common-extensions \
 && rm -rf /var/lib/apt/lists/*

# Build ROS 2 IASDK packages
# RUN source /opt/ros/foxy/setup.bash && \
#    cd ~/iasdk-public/ros/ros2 && \
#    colcon build && \
#    echo "source ~/iasdk-public/ros/ros2/install/setup.bash" >> ~/.bashrc

# Final environment setup
ENV DEBIAN_FRONTEND=dialog

#####################
# Development image #
#####################
FROM base AS dev

ARG CATKIN_WORKSPACE_DIR="/catkin_ws"
ARG USERNAME="developer"
ARG UID=1000
ARG GID=1000

WORKDIR ${CATKIN_WORKSPACE_DIR}

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
 && apt-get install -y \
    ack \
    bmon \
    cloc \
    gdb \
    htop \
    iperf3 \
    iputils-ping \
    mlocate \
    net-tools \
    psmisc \
    tmux \
    xterm \
 && rm -rf /var/lib/apt/lists/*

RUN apt-get update \
 && apt-get install -y \
    python3-vcstool \
    ros-${ROS_DISTRO}-rqt-common-plugins \
    ros-${ROS_DISTRO}-rqt-robot-dashboard \
    ros-${ROS_DISTRO}-rviz2 \
 && rm -rf /var/lib/apt/lists/*

RUN apt-get update \
 && apt-get install -y sudo \
 && rm -rf /var/lib/apt/lists/* \
 && addgroup --gid ${GID} ${USERNAME} \
 && adduser --disabled-password --gecos '' --uid ${GID} --gid ${GID} ${USERNAME} \
 && echo ${USERNAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USERNAME} \
 && chown -R ${UID}:${GID} /home/${USERNAME} \
 && chown -R ${UID}:${GID} ${CATKIN_WORKSPACE_DIR}

ENV DEBIAN_FRONTEND=dialog

RUN echo "alias rsource='source ${CATKIN_WORKSPACE_DIR}/devel/setup.bash'" >> /home/${USERNAME}/.bash_aliases \
 && echo "alias rbuild='(cd ${CATKIN_WORKSPACE_DIR} && catkin build)'" >> /home/${USERNAME}/.bash_aliases \
 && echo "alias rclean='(cd ${CATKIN_WORKSPACE_DIR} && catkin clean -y)'" >> /home/${USERNAME}/.bash_aliases \
 && echo "rsource || source /opt/ros/${ROS_DISTRO}/setup.bash" >> /home/${USERNAME}/.bashrc

USER ${USERNAME}
