
# EvaRobot-Kinetic
This explanation is how to apply to upgraded version of **"evapi_ros"** Packages to **Kinetic ROS**  (ROBOT Side Packages)

## Getting Started
Personally, I recommend you to run commands one by one, thanks to thats way you can see the source of error and upgrade the Evarobot packages to **any version of Linux or ROS**. if you see error, please do not hesitate to create new issue.

if you prefer to use prepared image of SD Card, contact with me.

To access to old [Indigo version of EVAPI_ROS](https://github.com/inomuh/evapi_ros)
## Preparation

### Take of SD Card


----------


### Configure Mikrotik Router

----------

## Installation

### Preparation Of Rasbian (To SD Card)
Install Lite Debian to SD card
* [Download](https://www.raspberrypi.org/downloads/raspbian/) - Select Lite version
* [Installing](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)

For use Raspberry PI 2 **headless**, create an empty file with name 'ssh' to boot directory of SD Card and turn on robot after insert SD Card to PI2.

----------

### Build Ros
* [Instructions](http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Kinetic%20on%20the%20Raspberry%20Pi)
```bash
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
```

----------

###  Build Evapi_modules
* [Instructions](https://github.com/inomuh/evapi_modules)

However offical version of evapi_modules **need patch** due to new kernel of linux and new modules.
```bash
cd ~
git clone https://github.com/inomuh/evapi_modules.git
```

**Apply Patches**

```bash
wget https://raw.githubusercontent.com/akahmet/EvaRobot-Kinetic/master/evapi_modules.patch
patch -p0 -R < evapi_modules.patch
```

***Continue...***

```bash
sudo rpi-update
sudo wget https://raw.githubusercontent.com/notro/rpi-source/master/rpi-source -O /usr/bin/rpi-source
sudo chmod +x /usr/bin/rpi-source
/usr/bin/rpi-source -q --tag-update
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
```

Run that command on ```sudo nano /etc/rc.local``` and add these lines:
```bash
	sudo bash /home/pi/evapi_modules/module_sonar/./setup_sonar_driver.sh 
	sudo bash /home/pi/evapi_modules/module_encoder/./setup_encoder_driver.sh
	sudo bash /home/pi/evapi_modules/module_bumper/./setup_bumper_driver.sh
```
***Note: To exit from nano press ctrl+x***
```sudo nano /etc/modules``` same as before 
```
	snd-bcm2836
	i2c-dev
	i2c-bcm2709
	driver_encoder
	driver_sonar
	driver_bumper
```

----------

### Build dependencies of evapi_ros
```bash
cd ~
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src 
catkin_init_workspace
cd ~/catkin_ws
catkin_make
source devel/setup.bash
echo "source /home/pi/catkin_ws/devel/setup.bash" >> ~/.bashrc
```

***Build Poco***
```bash
cd /tmp
git clone https://github.com/pocoproject/poco.git
cd /tmp/poco
mkdir cmake_build
cd cmake_build
cmake ..
make
sudo make install
sudo apt-get install libtinyxml-dev
```
***Continue... Be carefull about the compilation order.***
```bash
cd ~/catkin_ws/src
git clone https://github.com/ros/bond_core.git
git clone https://github.com/ros/class_loader.git
git clone https://github.com/ros/pluginlib.git
git clone https://github.com/ros/common_msgs.git
git clone https://github.com/ros-controls/realtime_tools.git
git clone https://github.com/inomuh/im_msgs.git 
cd ~/catkin_ws
catkin_make
cd ~/catkin_ws/src
git clone https://github.com/ros/dynamic_reconfigure.git
git clone https://github.com/ros/diagnostics.git	
cd ~/catkin_ws
catkin_make
```

----------

###  Build evapi_ros
* [Instructions](https://github.com/inomuh/evapi_ros)

```bash
cd ~/catkin_ws/src
git clone https://github.com/inomuh/evapi_ros.git
cd ~/catkin_ws
catkin_make
```

## Authors

* **Ahmet AK** - *Initial work* - [inomuh](https://github.com/inomuh)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This documentation is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* [Eva Robot](https://github.com/inomuh)
* Raspberry pi 2
