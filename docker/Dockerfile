FROM ros:melodic-robot

LABEL Description="Customized ROS-Melodic-Robot for CoppeliaSim 4.0.0 in Ubuntu 18.04" Version="1.0"

# Arguments
ARG user=user
ARG uid=1000
ARG shell=/bin/bash
ARG copp_ws="/home/${user}"

# define ros distribution version
ARG ROS_DISTRO=melodic
ARG PROJECT=RTOS_CAPP

# 
# Leave as empty string "" if you don't want any nvidia drivers to be installed.
# Otherwise, manually set the link to download the NVidia driver model for your system,
# in Linux x86_x64 version.
# This link can be found here:
#
# https://www.nvidia.com/Download/index.aspx?lang=en-us
#
# Example: for a GeForce RTX 2080 Ti running under a Windows x64, the Linux x86_64 version is:
# nvidia_driver_link=
# "http://us.download.nvidia.com/XFree86/Linux-x86_64/410.66/NVIDIA-Linux-x86_64-410.66.run"
#
# ARG nvidia_driver_link="http://uk.download.nvidia.com/XFree86/Linux-x86_64/430.26/NVIDIA-Linux-x86_64-430.26.run"
ARG nvidia_driver_link=""
						  

#
# ========================== Install required and useful packages ==========================
#
RUN printf '\n\n Installing Required Packages.. \n\n'

RUN apt-get update -y && apt-get install --no-install-recommends -y \
 xz-utils lsb-release apt-utils mesa-utils build-essential \
 software-properties-common locales x11-apps \
 git \
 xvfb \
 gedit gedit-plugins nano vim \
 zsh screen tree \
 sudo ssh synaptic \
 wget curl unzip htop \
 gdb valgrind \
 libcanberra-gtk* \
 python-keybinder python-notify \
 python-tempita python-lxml default-jre xsltproc \
 python-setuptools \
 python-pip\
 python3-pip\ 
 python-rosinstall \
 python-rosinstall-generator \
 python-wstool build-essential \
 ros-melodic-catkin python-catkin-tools \
 ros-melodic-moveit ros-melodic-moveit-plugins \ 
 ros-melodic-moveit-planners \ 
&& pip3 install numpy \
&& rm -rf /var/lib/apt/lists/* \
&& apt-get clean


#
# ========================== Set Timezone and New User ==========================
#

## Configure timezone and locale
#RUN sudo locale-gen en_US.UTF-8  
#ENV LANG en_US.UTF-8  
#ENV LANGUAGE en_US:en  
#ENV LC_ALL en_US.UTF-8

RUN printf '\n\n Setting New User.. \n\n'

# Crete and add user
ENV USER=${user}

RUN useradd -ms ${shell} ${user} \
&& export uid=${uid} gid=${uid} \
&& echo "${user} ALL=(ALL) NOPASSWD: ALL" > "/etc/sudoers.d/${user}" && \
chmod 0440 "/etc/sudoers.d/${user}"

# Switch to user
USER ${user}


#
# ========================== ROS Workspace Setup ==========================
#

#RUN sudo rosdep init
RUN sudo rosdep fix-permissions && rosdep update  \
    && echo "source "/opt/ros/${ROS_DISTRO}/setup.bash"" >> /home/${user}/.bashrc
    
RUN mkdir -p /home/${user}/catkin_ws/$PROJECT/base/src && \
mkdir /home/${user}/catkin_ws/$PROJECT/base/logs

RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> $HOME/.bashrc

RUN /bin/bash -c '. /opt/ros/$ROS_DISTRO/setup.bash; catkin_init_workspace /home/${user}/catkin_ws/$PROJECT/base/src'
RUN /bin/bash -c '. /opt/ros/$ROS_DISTRO/setup.bash; cd /home/${user}/catkin_ws/$PROJECT/base/; catkin init'
RUN /bin/bash -c '. /opt/ros/$ROS_DISTRO/setup.bash; cd /home/${user}/catkin_ws/$PROJECT/base/; catkin build'
RUN echo "source /home/${user}/catkin_ws/$PROJECT/base/devel/setup.bash" >> /home/${user}/.bashrc

RUN sudo rm -rf /home/${user}/catkin_ws/$PROJECT/base/src/*.txt 



#
# ========================== Download and Install V-REP  ==========================
#

RUN printf '\n\n Fechting and Installing V-REP.. \n\n'

##Installation CoppeliaSim 4.0.0: Download CoppeliaSim from link (may be outdated) and then Install
RUN wget -P ${copp_ws} http://www.coppeliarobotics.com/files/CoppeliaSim_Edu_V4_0_0_Ubuntu18_04.tar.xz \
&& tar -C ${copp_ws} -xpJf ${copp_ws}/CoppeliaSim_Edu_V4_0_0_Ubuntu18_04.tar.xz \
&& sudo apt-get update -y && sudo apt-get install --no-install-recommends -y \
libgl1-mesa-dev libavcodec-dev libavformat-dev libswscale-dev libopencv*

##Installation VREP 3.6.1: Download V-REP from link (may be outdated) and then Install
# RUN wget -P ${vrep_ws} http://coppeliarobotics.com/files/V-REP_PRO_EDU_V3_6_1_Ubuntu16_04.tar.xz \
# && tar -C ${vrep_ws} -xf ${vrep_ws}/V-REP_PRO_EDU_V3_6_1_Ubuntu16_04.tar.xz -v

#
# ========================== Install NVidia Driver, if desired ==========================
#

RUN if [ "$nvidia_driver_link" != "" ]; then \
	printf '\n\n Installing Nvidia Driver, as requested.. \n\n' \	
	&& export nvidia_driver=$(echo $nvidia_driver_link | rev | cut -d'/' -f 1 | rev) \
	#Install required packages..
	&& sudo apt-get update -y && sudo apt-get install --no-install-recommends -y mesa-utils binutils kmod \
	#Download NVidia chosen driver
	&& wget -P /tmp/ ${nvidia_driver_link} \
	#Install the driver previously downloaded
        && chmod +x /tmp/${nvidia_driver} \
	&& sudo sh /tmp/${nvidia_driver} -a --ui=none --no-kernel-module \
	&& rm /tmp/${nvidia_driver} \
;fi
#
# ========================== Final Installations ==========================
#

RUN printf '\n\n Runnning Final Update.. \n\n'

# Run a final update-upgrade routines
RUN sudo apt-get update -y \
&& sudo apt-get dist-upgrade -y


#
# ========================== Final environment configs ==========================
#

# Reset APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE to default value
ENV APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=0

# Create a mount point to bind host data to
# to check where the volume is mounted in the host machine:
# "docker inspect -f "{{json .Mounts}}" vol-test | jq ."
VOLUME /external

# Make SSH available
EXPOSE 22

# This is required for sharing Xauthority
ENV QT_X11_NO_MITSHM=1

# Set the starting working directory.
WORKDIR /

# Set the image to start opening a new bash terminal
ENTRYPOINT ["/bin/bash"]
