# EvaRobot-Kinetic

## Prerequisites

### PREAPATION OF RASBIAN 
install base debian to sd card
[download](https://www.raspberrypi.org/downloads/raspbian/) - Select lite version
[installing](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)

for use raspberry pi headless create empty ssh file to boot directory of raspberry pi

after of this build ros from source
http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Kinetic%20on%20the%20Raspberry%20Pi
/* BUILD ROS */
sudo apt-get update
sudo apt-get install -y python-rosdep python-rosinstall-generator python-wstool python-rosinstall build-essential cmake
sudo rosdep init
rosdep update
mkdir -p ~/ros_catkin_ws
cd ~/ros_catkin_ws
rosinstall_generator ros_comm --rosdistro kinetic --deps --wet-only --tar > kinetic-ros_comm-wet.rosinstall
wstool init src kinetic-ros_comm-wet.rosinstall
wstool update -j4 -t src
mkdir -p ~/ros_catkin_ws/external_src
cd ~/ros_catkin_ws/external_src
wget http://sourceforge.net/projects/assimp/files/assimp-3.1/assimp-3.1.1_no_test_models.zip/download -O assimp-3.1.1_no_test_models.zip
unzip assimp-3.1.1_no_test_models.zip
cd assimp-3.1.1
cmake .
make
sudo make install
cd ~/ros_catkin_ws
rosdep install --from-paths src --ignore-src --rosdistro kinetic -y
sudo ./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic -j2
source /opt/ros/kinetic/setup.bash
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc


/* BUILD EVA MODULES */
https://github.com/inomuh/evapi_modules
cd ~
git clone https://github.com/inomuh/evapi_modules.git
sudo rpi-update
sudo wget https://raw.githubusercontent.com/notro/rpi-source/master/rpi-source -O /usr/bin/rpi-source && sudo chmod +x /usr/bin/rpi-source && /usr/bin/rpi-source -q --tag-update
rpi-source --skip-gcc
cd /lib/modules/$(uname -r)/kernel/drivers/
sudo mkdir evarobot
cd ~/evapi_modules/module_sonar
make
sudo cp driver_sonar.ko /lib/modules/$(uname -r)/kernel/drivers/evarobot/
cd ~/evapi_modules/module_encoder
make
sudo cp driver_encoder.ko /lib/modules/$(uname -r)/kernel/drivers/evarobot/
cd ~/evapi_modules/module_bumper
make
sudo cp driver_bumper.ko /lib/modules/$(uname -r)/kernel/drivers/evarobot/
cd /lib/modules/$(uname -r)/kernel/drivers/evarobot/
sudo insmod driver_encoder.ko
sudo insmod driver_sonar.ko
sudo insmod driver_bumper.ko

sudo nano /etc/rc.local
	sudo bash /home/pi/evapi_modules/module_sonar/./setup_sonar_driver.sh 
	sudo bash /home/pi/evapi_modules/module_encoder/./setup_encoder_driver.sh
	sudo bash /home/pi/evapi_modules/module_bumper/./setup_bumper_driver.sh

sudo nano /etc/modules
	snd-bcm2836
	i2c-dev
	i2c-bcm2709
	driver_encoder
	driver_sonar
	driver_bumper

/* BUILD EVAPI  */
https://github.com/inomuh/evapi_ros
cd ~
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src 
catkin_init_workspace
cd ~/catkin_ws
catkin_make
source devel/setup.bash
echo "source /home/pi/catkin_ws/devel/setup.bash" >> ~/.bashrc

cd ~/catkin_ws/src
git clone https://github.com/ros/diagnostics.git
git clone https://github.com/ros/dynamic_reconfigure.git
git clone https://github.com/inomuh/im_msgs.git 
git clone https://github.com/ros/common_msgs.git
cd ~/catkin_ws
catkin_make
cd ~/catkin_ws/src
git clone https://github.com/inomuh/evapi_ros.git
cd ~/catkin_ws
catkin_make

git clone https://github.com/inomuh/evapi_ros.git

cd ~/catkin_ws
catkin_make