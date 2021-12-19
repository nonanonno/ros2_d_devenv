ARG base_image="nvidia/cudagl:11.4.2-devel-ubuntu20.04"

FROM ${base_image} as cudagl

ENV DEBIAN_FRONTEND noninteractive
ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics

# upgrade image
RUN apt-get update && apt-get upgrade -y && rm -rf /var/lib/apt/lists/*

ARG ROS_DISTRO=foxy
ENV ROS_DISTRO=${ROS_DISTRO}
ENV LANG en_US.UTF-8

# install ros
RUN apt-get update && apt-get install -y \
    curl \
    gnupg2 \
    lsb-release \
    locales \
    && locale-gen en_US en_US.UTF-8 \
    && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 \
    && curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | apt-key add - \
    && sh -c 'echo "deb [arch=$(dpkg --print-architecture)] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2-latest.list' \
    && apt-get update && apt-get install -y \
    ros-${ROS_DISTRO}-desktop \
    python3-colcon-common-extensions \
    python3-rosdep \
    python3-argcomplete \
    python3-vcstool \
    && rosdep init && rosdep update \
    && rm -rf /var/lib/apt/lists/

# install dmd
ARG dmd_version="2.098.0"
RUN apt-get update && apt-get install -y wget \
    && wget http://downloads.dlang.org/releases/2.x/${dmd_version}/dmd_${dmd_version}-0_amd64.deb -O dmd.deb \
    && dpkg -i dmd.deb || true \
    && apt --fix-broken install -y \
    && rm -rf /var/lib/apt/lists/*

# for dpp
RUN apt-get update && apt-get install -y \
    clang-10 \
    libclang-10-dev \
    && rm -rf /var/lib/apt/lists/* \
    && update-alternatives --install /usr/bin/clang clang /usr/bin/clang-10 1

# other tools
RUN apt-get update && apt-get install -y \
    git \
    vim \
    bash-completion  \
    python3-pip \
    python3-setuptools \
    && rm -rf /var/lib/apt/lists/*


# add user
ARG username=developer
ARG user_uid=1000
ARG user_gid=${user_uid}
RUN groupadd --gid ${user_gid} ${username} \
    && useradd --uid ${user_uid} --gid ${user_gid} -m ${username} -s /bin/bash \
    && apt-get update \
    && apt-get install -y sudo \
    && echo ${username} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${username} \
    && chmod 0440 /etc/sudoers.d/${username}

# set bashrc
USER ${username}
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> $HOME/.bashrc \
    && echo "export TERM=xterm-256color" >> $HOME/.bashrc \
    && echo "source /etc/bash_completion.d/git-prompt" >> $HOME/.bashrc \
    && echo "GIT_PS1_SHOWDIRTYSTATE=true" >> $HOME/.bashrc \
    && echo "export PS1='\[\033[01;32m\]\u@\h:\[\033[01;34m\]\w\[\033[00m\] (ros-${ROS_DISTRO}) (dmd-${dmd_version})"'\[\033[32m\]$(__git_ps1)\[\033[00m\]'"\n\$ '" >>  $HOME/.bashrc

# setup entrypoint
USER root
COPY ./ros_entrypoint.sh /
RUN chmod +x /ros_entrypoint.sh

ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]