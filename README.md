#**REPO HAS BEEN MOVED TO DOTROBOT FOLDER.**


# P12 ROS
 Temporary repository for the ros based controller. Move to a DotRobot owned repo once someone has time/feels like it
# Welcome to the P12 repository
This repository is maintained by Pascal Christiaanse (Intern, will be back 22nd aug 2022). <br>
You can reach me during my vacation via whatsapp on 0655289809, although ill probably take a while to answer.



# Quick start
The quick start shows how to run what has currently been made and tested. It does not allow to change significant details about the control system. For this; follow the setup guide below. 

## Step 1: Setup environment
1. Ensure you are running a Linux environment, specifically Ubuntu 20.04 LTS. This environment may be run from a VM (for example, use VirtualBox. Make sure to allocate atleast ~55Gb)
2. Install VSCode<br>
   ```sh
   sudo apt-get install code
   ```

## Step 2: Install ROS2: Galactic 
Install ROS2: Galactic according to these instructions: [ROS2: Galactic setup](https://docs.ros.org/en/galactic/Installation/Alternatives/Ubuntu-Development-Setup.html) or go to: docs.ros.org -> "ROS2 galactic" -> "Installation" -> "Alternatives" -> "Ubuntu (source)". <br>
Follow the steps to atleast "Environment setup -> Source the setup script"
> Optional: You can add the /ros2_galactic/install/setup.sh to your /.bashrc file to automatically source the ros install when starting a terminal. You can do this by typing ```code ~/.bashrc``` in terminal and adding ```. ~/ros2_galactic/install/setup.sh``` at the end of the file

## Step 3: Install and run MicroROS 
For reference, you can visit [The microRos documentation](https://micro.ros.org/docs/tutorials/core/first_application_rtos/freertos/). 
1. **Clone and build micro_ros_setup <br>**
    Source the ROS 2 installation
    ```sh
    source /ros2_galactic/install/setup.sh
    ```
    Create a workspace and download the micro-ROS tools
    ```sh 
    mkdir P12_ROS/microros_ws
    cd P12_ROS/microros_ws
    git clone -b $ROS_DISTRO https://github.com/micro-ROS/micro_ros_setup.git src/micro_ros_setup
    ```
    Update dependencies using rosdep
    ```sh
    sudo apt update && rosdep update
    rosdep install --from-paths src --ignore-src -y
    ```
    Install pip
    ```sh
    sudo apt-get install python3-pip
    ```
    Build micro-ROS tools and source them
    ```sh
    colcon build
    source install/local_setup.bash
    ```
    >Optional: add `. ~/P12_ROS/microros_ws/install/local_setup.bash` to ~/.bashrc
2. **Creating the micro-ROS agent**
   Create a micro_ros_agent workspace in P12 root
   ```sh
   mkdir ~/P12_ROS/micro_ros_agent_ws
   cd ~/P12_ROS/micro_ros_agent_ws
   ros2 run micro_ros_setup create_agent_ws.sh
   ```
   Build the workspace using micro_ros_setup
   ```sh
   ros2 run micro_ros_setup build_agent.sh
   source install/local_setup.bash
   ```
   >Optional: add `. ~/P12_ROS/micro_ros_agent_ws/install/local_setup.bash` to ~/.bashrc
3. **Running the micro-ROS agent<br>**
   Assuming the only connected USB devices are the 4 boards; if this is not the case, identify which usb ports are owned by the esp boards with either `dmesg` or `lsusb`
   ```sh
   ros2 run micro_ros_agent micro_ros_agent serial --devs '/dev/ttyUSB0 /dev/ttyUSB1 /dev/ttyUSB2 /dev/ttyUSB3'
   ```
   ***This program needs to run at all times as this exposes the ROS nodes on te board to the DDS world. Failing to do so will mean the topics will not be available.***<br>
   Next; Reset all boards one by one. There should be a flow of information somewhere along the lines of 
   ```sh
    [1659086544.413371] info     | Root.cpp           | create_client            | create                 | client_key: 0x41E79D19, session_id: 0x81
    [1659086544.413377] info     | SessionManager.hpp | establish_session        | session established    | client_key: 0x41E79D19, address: 5
    [1659086544.428077] info     | ProxyClient.cpp    | create_participant       | participant created    | client_key: 0x41E79D19, participant_id: 0x000(1)
    [1659086544.447186] info     | ProxyClient.cpp    | create_topic             | topic created          | client_key: 0x41E79D19, topic_id: 0x000(2), participant_id: 0x000(1)
    [1659086544.458091] info     | ProxyClient.cpp    | create_publisher         | publisher created      | client_key: 0x41E79D19, publisher_id: 0x000(3), participant_id: 0x000(1)
    [1659086544.469269] info     | ProxyClient.cpp    | create_datawriter        | datawriter created     | client_key: 0x41E79D19, datawriter_id: 0x000(5), publisher_id: 0x000(3)
    [1659086544.486986] info     | ProxyClient.cpp    | create_topic             | topic created          | client_key: 0x41E79D19, topic_id: 0x001(2), participant_id: 0x000(1)
    [1659086544.497094] info     | ProxyClient.cpp    | create_publisher         | publisher created      | client_key: 0x41E79D19, publisher_id: 0x001(3), participant_id: 0x000(1)
    [1659086544.508159] info     | ProxyClient.cpp    | create_datawriter        | datawriter created     | client_key: 0x41E79D19, datawriter_id: 0x001(5), publisher_id: 0x001(3)
    [1659086544.527095] info     | ProxyClient.cpp    | create_topic             | topic created          | client_key: 0x41E79D19, topic_id: 0x002(2), participant_id: 0x000(1)
    [1659086544.537916] info     | ProxyClient.cpp    | create_subscriber        | subscriber created     | client_key: 0x41E79D19, subscriber_id: 0x000(4), participant_id: 0x000(1)
    [1659086544.550303] info     | ProxyClient.cpp    | create_datareader        | datareader created     | client_key: 0x41E79D19, datareader_id: 0x000(6), subscriber_id: 0x000(4)
    ```
4. **Testing (playing)<br>**
    The control topics should now be accessible. You can check this by running `ros2 topic list`. You can now control the robot using by running `rqt` and publishing to the topic "[...]/set_tgt_pos"<br>
    General speeds and range limits: <br>
    * X-axis: 
        * Position: 0 - ~160 $rads$
        * Velocity: Max 25 $rads/s$
        * Acceleration: Max 100 $rads/s^2$
    * Y-axis: 
        * Position: 0 - ~55 $rads$
        * Velocity: Max 15 $rads/s$
        * Acceleration: Max 50 $rads/s^2$
    * Z-axis: 
        * Position: 0 - ~62 $rads$
        * Velocity: Max 50 $rads/s$
        * Acceleration: Max 250 $rads/s^2$ <br>
    >Be aware that these are far from the maximum speeds, but they have been tested and are known to be safe.
## Step 4: Get the control code
To run the sequences programmed so far, you will need to build the "robot_controller" package
**1. Create a workspace for pure ROS packages <br>**
```sh
mkdir ~/P12_ROS/ROS/src
cd ~/P12_ROS/ROS/src
```
**2. Get the control package<br>**
Now move the folders "robot_controller", "stepper_commands" and "magnet_message" into the src folder you just created
3. Build and source the workspace
```sh
cd ~/P12_ROS/ROS
colcon build
source ~/P12_ROS/ROS/install/setup.sh
```
**3. Run the executable**
```sh
ros2 run robot_controller test_suite
```
> When working on this package, you can also just run it as a regular python file once it has been built once

## **Remarks** <br>
For now, only the "Tester" program should be run. The other programs have *questionable* architectures. The Tester program is somewhat more thought through.<br>
Feel free to run them and see what happens, but when making new programs, the Tester architecture should be good enough of an example


