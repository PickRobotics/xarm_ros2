# xarm_ros2

## 1. 소개

&ensp;&ensp;&ensp;&ensp; 이 저장소는 UFACTORY의 xArm 시리즈의 시뮬레이션 모델과 이에 대응하는 모션 플래닝 및 제어 데모를 포함하고 있습니다. 개발 및 테스트 환경은 다음과 같습니다.
- Ubuntu 22.04 + ROS Humble


&ensp;&ensp;&ensp;&ensp;사용하는 환경에 따라서 관련 코드로 브랜치하세요.
- Humble: [humble](https://github.com/xArm-Developer/xarm_ros2/tree/humble)

## 2. 업데이트 History    
- moveit dual arm control (under single rviz GUI), each arm can be separately configured（e.g. DOF, add_gripper, etc）
- add support for Gazebo simulation, can be controlled by moveit.
- support adding customized tool model.  
- (2022-09-07) Change the parameter type of service (__set_tgpio_modbus_timeout__/__getset_tgpio_modbus_data__), and add parameters to support transparent transmission
- (2022-09-07) Change topic name (xarm_states to robot_states)
- (2022-09-07) Update submodule xarm-sdk to version 1.11.0
- (2022-09-09) [Beta]Support Humble version
- (2022-10-10) xarm_api adds some services
- (2022-12-15) Add parameter `add_realsense_d435i` to load RealSense D435i camera model and support gazebo simulation
- (2023-03-29) Added the launch parameter `model1300` (default is false), and replaced the model of the end of the xarm robot arm with the 1300 series
- (2023-04-20) Update the URDF file, adapt to ROS1 and ROS2, and load the inertia parameters of the link from the configuration file according to the SN
- (2023-04-20) Added the launch parameter `add_d435i_links` (default is false), which supports adding the link relationship between D435i cameras when loading the RealSense D435i model. It is only useful when `add_realsense_d435i` is true
- (2023-04-20) Lite6 supports `add_realsense_d435i` and `add_d435i_links` parameters
- (2023-04-20) Added the launch parameter `robot_sn`, supports loading the inertia parameters of the corresponding joint link, and automatically overrides the `model1300` parameters
- (2023-04-20) Added launch parameters `attach_to`/`attach_xyz`/`attach_rpy` to support attaching the robot arm model to other models
- (2023-06-07) Added support for UFACTORY850 robotic arm
- (2023-10-12) Added the generation and use of joint kinematics parameter files
- (2024-01-17) Added support for xarm7_mirror model robotic arm
- (2024-02-27) Added support for Bio Gripper (parameter `add_bio_gripper`, Lite6 is not supported)


## 3. 준비

- ### 3.1 [ROS2 설치](https://docs.ros.org/) 
  - [Humble](https://docs.ros.org/en/ros2_documentation/humble/Installation.html)

- ### 3.2 [Moveit2 설치](https://moveit.ros.org/install-moveit2/binary/)  

- ### 3.3 [Gazebo 설치](https://classic.gazebosim.org/tutorials?tut=install_ubuntu)  

- ### 3.4 [gazebo_ros_pkgs 설치](http://gazebosim.org/tutorials?tut=ros2_installing&cat=connect_ros)  

## 4. 사용 방법

- ### 4.1 workspace 생성
    ```bash
    $ cd ~
    $ mkdir -p dev_ws/src
    ```

- ### 4.2 "xarm_ros2" 저장소에서 소스코드 가져오기
    ```bash
    # Remember to source ros2 environment settings first
    $ cd ~/dev_ws/src
    # DO NOT omit "--recursive"，or the source code of dependent submodule will not be downloaded.
    # Pay attention to the use of the -b parameter command branch, $ROS_DISTRO indicates the currently activated ROS version, if the ROS environment is not activated, you need to customize the specified branch (foxy/galactic/humble)
    $ git clone https://github.com/xArm-Developer/xarm_ros2.git --recursive -b $ROS_DISTRO
    ```

- ### 4.3 "xarm_ros2" 저장소 업데이트
    ```bash
    $ cd ~/dev_ws/src/xarm_ros2
    $ git pull
    $ git submodule sync
    $ git submodule update --init --remote
    ```

- ### 4.4 의존성 설치
    ```bash
    # Remember to source ros2 environment settings first
    $ cd ~/dev_ws/src/
    $ rosdep update
    $ rosdep install --from-paths . --ignore-src --rosdistro $ROS_DISTRO -y
    ```

- ### 4.5 xarm_ros2 빌드하기
    ```bash
    # Remember to source ros2 and moveit2 environment settings first
    $ cd ~/dev_ws/
    # build all packages
    $ colcon build
    
    # build selected packages
    $ colcon build --packages-select xarm_api
    ```


## 5. Package 소개

__Reminder 1: 같은 LAN 환경에서 여러 사람이 ROS 2를 사용하는 경우, ROS_DOMAIN_ID__ 설정하기
  - [Humble](https://docs.ros.org/en/ros2_documentation/humble/Concepts/About-Domain-ID.html)

__Reminder 2： xarm_ros2 내에 있는 application을 실행하기 전에 환경 설정 script를 source 하기__

```bash
$ cd ~/dev_ws/
$ source install/setup.bash
```
__Reminder 3： 모든 다음 명령은 xArm6를 기반으로 하며, xArm5와 xArm7에 대해서는 적절한 parameters와 파일 이름을 사용해야 합니다__
__Reminder 4: 아래 <hw_ns>는 실제 값으로 대체되며, xarm 시리즈 기본값은 xarm이며 나머지 기본값은 ufactory입니다.__


- ### 5.1 xarm_description
    이 package는 robot description와 xArm의 3D model을 포함하고 있습니다. 모델은 다음 launch 파일로 RViz내에 나타나게 할 수 있습니다.:
    ```bash
    $ cd ~/dev_ws/
    # set 'add_gripper=true' to attach xArm gripper model
    # set 'add_vacuum_gripper=true' to attach xArm vacuum gripper model
    # 주의：한개의 end_effector만 부착한 경우('true'로 설정).
    $ ros2 launch xarm_description xarm6_rviz_display.launch.py [add_gripper:=true] [add_vacuum_gripper:=true]
    ```

- ### 5.2 xarm_msgs  
    이 package는 xarm_ros2에 대한 모든 interface 정의를 포함하며, 이를 사용하기 전에 파일 내에 있는 명령들을 확인하세요. [README](./xarm_msgs/ReadMe.md)

- ### 5.3 xarm_sdk
    이 package는 이 프로젝트의 submodule로 제공되며, 관련 git 저장소는 [xArm-CPLUS-SDK](https://github.com/xArm-Developer/xArm-CPLUS-SDK) 이다. 실제 xArms과 인터페이스를 위해서, "xArm-CPLUS-SDK" 내에 있는 문서를 참고한다.

- ### 5.4 xarm_api
    이 package는 "xarm_sdk"의 ros wrapper이며 함수들은 ros service나 ros topic으로 구현됩니다. "xarm_ros2" 내에 실제 xArm과 통신은 이 파트에서 제공되는 services와 topics을 기반으로 이뤄집니다. 모든 services와 topics은 <hw_ns>/ namespace 아래에 있습니다.(예 : "joint_states"에 대한 전체 이름은 "<hw_ns>/joint_states" 입니다.)
    
    - __services__: 제공하는 service의 이름은 SDK에 있는 관련 이름과 동일합니다. 하지만 service를 활성화시키는 것은 "services" 도메인 아래에 있는 설정 파일인 ```xarm_api/config/xarm_params.yaml``` 과 ```xarm_api/config/xarm_user_params.yaml```을 따릅니다. 정의한 service는 해당 service가 ```true```로 설정되어 있는 경우에만 초기화 시점에 활성화됩니다. 만약 parameters를 커스텀해야 하는 경우, ```xarm_api/config/xarm_user_params.yaml``` 파일을 생성하여 수정하며 ```xarm_api/config/xarm_params.yaml```을 참고하세요.
        ```
        services:
            motion_enable: true
            set_mode: true
            set_state: true
            clean_conf: false
            ...
        ```

    - __topics__:  

        __joint_states__: is of type __sensor_msgs::msg::JointState__  

        __robot_states__: is of type __xarm_msgs::msg::RobotMsg__  

        __xarm_cgpio_states__: is of type __xarm_msgs::msg::CIOState__  

        __uf_ftsensor_raw_states__: is of type __geometry_msgs::msg::WrenchStamped__  

        __uf_ftsensor_ext_states__: is of type __geometry_msgs::msg::WrenchStamped__  

        __Note:__: topics 중에 일부는 __report_type__ 이 launch 단계에서 설정되어 있는 경우에만 유효합니다. 참고 [here](https://github.com/xArm-Developer/xarm_ros#report_type-argument).  

    
    - __Launch and test (xArm)__:  

        ```bash
        $ cd ~/dev_ws/
        # launch xarm_driver_node
        $ ros2 launch xarm_api xarm6_driver.launch.py robot_ip:=192.168.1.117
        # service test
        $ ros2 run xarm_api test_xarm_ros_client
        # topic test
        $ ros2 run xarm_api test_robot_states
        ```

    - __Use command line (xArm)__:

        ```bash
        $ cd ~/dev_ws/
        # launch xarm_driver_node:
        $ ros2 launch xarm_api xarm6_driver.launch.py robot_ip:=192.168.1.117
        
        # enable all joints:
        $ ros2 service call /xarm/motion_enable xarm_msgs/srv/SetInt16ById "{id: 8, data: 1}"
        
        # set proper mode (0) and state (0)
        $ ros2 service call /xarm/set_mode xarm_msgs/srv/SetInt16 "{data: 0}"
        $ ros2 service call /xarm/set_state xarm_msgs/srv/SetInt16 "{data: 0}"
        
        # Cartesian linear motion: (unit: mm, rad)
        $ ros2 service call /xarm/set_position xarm_msgs/srv/MoveCartesian "{pose: [300, 0, 250, 3.14, 0, 0], speed: 50, acc: 500, mvtime: 0}"   
        
        # joint motion for xArm6: (unit: rad)
        $ ros2 service call /xarm/set_servo_angle xarm_msgs/srv/MoveJoint "{angles: [-0.58, 0, 0, 0, 0, 0], speed: 0.35, acc: 10, mvtime: 0}"
        ```
    
    - __Use command line (lite6)__:

        ```bash
        $ cd ~/dev_ws/
        # launch ufactory_driver_node:
        $ ros2 launch xarm_api lite6_driver.launch.py robot_ip:=192.168.1.161
        
        # enable all joints:
        $ ros2 service call /ufactory/motion_enable xarm_msgs/srv/SetInt16ById "{id: 8, data: 1}"
        
        # set proper mode (0) and state (0)
        $ ros2 service call /ufactory/set_mode xarm_msgs/srv/SetInt16 "{data: 0}"
        $ ros2 service call /ufactory/set_state xarm_msgs/srv/SetInt16 "{data: 0}"
        
        # Cartesian linear motion: (unit: mm, rad)
        $ ros2 service call /ufactory/set_position xarm_msgs/srv/MoveCartesian "{pose: [250, 0, 250, 3.14, 0, 0], speed: 50, acc: 500, mvtime: 0}"   
        
        # joint motion: (unit: rad)
        $ ros2 service call /ufactory/set_servo_angle xarm_msgs/srv/MoveJoint "{angles: [-0.58, 0, 0, 0, 0, 0], speed: 0.35, acc: 10, mvtime: 0}"
        ```
    
    - __Use command line (UFACTORY850)__:

        ```bash
        $ cd ~/dev_ws/
        # launch ufactory_driver_node:
        $ ros2 launch xarm_api uf850_driver.launch.py robot_ip:=192.168.1.181
        
        # enable all joints:
        $ ros2 service call /ufactory/motion_enable xarm_msgs/srv/SetInt16ById "{id: 8, data: 1}"
        
        # set proper mode (0) and state (0)
        $ ros2 service call /ufactory/set_mode xarm_msgs/srv/SetInt16 "{data: 0}"
        $ ros2 service call /ufactory/set_state xarm_msgs/srv/SetInt16 "{data: 0}"
        
        # Cartesian linear motion: (unit: mm, rad)
        $ ros2 service call /ufactory/set_position xarm_msgs/srv/MoveCartesian "{pose: [250, 0, 250, 3.14, 0, 0], speed: 50, acc: 500, mvtime: 0}"   
        
        # joint motion: (unit: rad)
        $ ros2 service call /ufactory/set_servo_angle xarm_msgs/srv/MoveJoint "{angles: [-0.58, 0, 0, 0, 0, 0], speed: 0.35, acc: 10, mvtime: 0}"
        ```

    Note: 실제 robot에서 테스트하기 전에 [Mode](https://github.com/xArm-Developer/xarm_ros#6-mode-change), state와 motion 명령을 공부하세요. Please note **xArm 시리즈와 Lite 6에서 제공하는 services는 다음 namespaces를 가지고 있습니다**.  

- ### 5.5 xarm_controller
    이 package는 ros2 기반 실제 xArm control에 대한 하드웨어 인터페이스를 정의합니다.  

    ```bash
    $ cd ~/dev_ws/
    # For xArm(xarm6 as example): xArm gripper model을 부착하기 위해서 'add_gripper=true'로 설정
    $ ros2 launch xarm_controller xarm6_control_rviz_display.launch.py robot_ip:=192.168.1.117 [add_gripper:=true]

    # For lite6: Lite6 gripper 모델에 대해서 'add_gripper=true'로 설정
    $ ros2 launch xarm_controller lite6_control_rviz_display.launch.py robot_ip:=192.168.1.161 [add_gripper:=true]
    
    # For UFACTORY850: xarm gripper model을 부착하기 위해서 'add_gripper=true'로 설정
    $ ros2 launch xarm_controller uf850_control_rviz_display.launch.py robot_ip:=192.168.1.181 [add_gripper:=true]
    ```

- ### 5.6 xarm_moveit_config
    이 package는 moveit에서 xArm/Lite6를 제어하기 위한 기능을 제공합니다.

    - 【simulated】moveit 실행, rviz에서 robot 제어하기.  

        ```bash
        $ cd ~/dev_ws/
        # For xArm(xarm6 as example): set 'add_gripper=true' to attach xArm gripper model
        $ ros2 launch xarm_moveit_config xarm6_moveit_fake.launch.py [add_gripper:=true]

        # For Lite6: set 'add_gripper=true' to attach Lite6 gripper model
        $ ros2 launch xarm_moveit_config lite6_moveit_fake.launch.py [add_gripper:=true]

        # For UFACTORY850: set 'add_gripper=true' to attach xarm gripper model
        $ ros2 launch xarm_moveit_config uf850_moveit_fake.launch.py [add_gripper:=true]
        ```
    
    - 【real arm】Launch moveit, controlling robot in rviz.  

        ```bash
        $ cd ~/dev_ws/
        # For xArm(xarm6 as example): set 'add_gripper=true' to attach xArm gripper model
        $ ros2 launch xarm_moveit_config xarm6_moveit_realmove.launch.py robot_ip:=192.168.1.117 [add_gripper:=true]

        # For Lite6: set 'add_gripper=true' to attach Lite6 gripper model
        $ ros2 launch xarm_moveit_config lite6_moveit_realmove.launch.py robot_ip:=192.168.1.161 [add_gripper:=true]

        # For UFACTORY850: set 'add_gripper=true' to attach xarm gripper model
        $ ros2 launch xarm_moveit_config uf850_moveit_realmove.launch.py robot_ip:=192.168.1.181 [add_gripper:=true]
        ```
    
    - 【Dual simulated】하나의 moveit process를 실행하고 하나의 rviz내에서 2개 xArms을 제어하기.  

        ```bash
        $ cd ~/dev_ws/
        # set 'add_gripper=true' to attach xArm gripper model
        # 'add_gripper_1': can separately decide whether to attach gripper for left arm，default for same value with 'add_gripper'
        # 'add_gripper_2': can separately decide whether to attach gripper for right arm，default for same value with 'add_gripper'
        # 'dof_1': can separately configure the model DOF of left arm，default to be the same DOF specified in filename.
        # 'dof_2': can separately configure the model DOF of right arm，default to be the same DOF specified in filename.
        
        # For xArm (xarm6 here):
        $ ros2 launch xarm_moveit_config dual_xarm6_moveit_fake.launch.py [add_gripper:=true]

        # For Lite6:
        $ ros2 launch xarm_moveit_config dual_lite6_moveit_fake.launch.py [add_gripper:=true]

        # For UFACTORY850:
        $ ros2 launch xarm_moveit_config dual_uf850_moveit_fake.launch.py [add_gripper:=true]
        ```
    
    - 【Dual real arm】하나의 moveit process를 실행하고, 하나의 rviz내에서 2개 xArms을 제어하기.

        ```bash
        $ cd ~/dev_ws/
        # 'robot_ip_1': IP address of left arm
        # 'robot_ip_2': IP address of right arm
        # set 'add_gripper=true' to attach xArm gripper model
        # 'add_gripper_1': can separately decide whether to attach gripper for left arm，default for same value with 'add_gripper'
        # 'add_gripper_2': can separately decide whether to attach gripper for right arm，default for same value with 'add_gripper'
        # 'dof_1': can separately configure the model DOF of left arm，default to be the same DOF specified in filename.
        # 'dof_2': can separately configure the model DOF of right arm，default to be the same DOF specified in filename.
        
        # For xArm (xarm6 here):
        $ ros2 launch xarm_moveit_config dual_xarm6_moveit_realmove.launch.py robot_ip_1:=192.168.1.117 robot_ip_2:=192.168.1.203 [add_gripper:=true]
        
        # For Lite6:
        $ ros2 launch xarm_moveit_config dual_lite6_moveit_realmove.launch.py robot_ip_1:=192.168.1.117 robot_ip_2:=192.168.1.203 [add_gripper:=true]

        # For UFACTORY850:
        $ ros2 launch xarm_moveit_config dual_uf850_moveit_realmove.launch.py robot_ip_1:=192.168.1.181 robot_ip_2:=192.168.1.182 [add_gripper:=true]
        ```

- ### 5.7 xarm_planner
    이 package는 moveit API를 통해 xArm(시뮬레이션 혹은 실제 arm)을 제어하기 위한 함수를 제공한다.  

    ```bash
    $ cd ~/dev_ws/
    # 【simulated xArm】launch xarm_planner_node
    $ ros2 launch xarm_planner xarm6_planner_fake.launch.py [add_gripper:=true]
    # 【real xArm】launch xarm_planner_node
    $ ros2 launch xarm_planner xarm6_planner_realmove.launch.py robot_ip:=192.168.1.117 [add_gripper:=true]

    # 【simulated Lite6】launch xarm_planner_node
    $ ros2 launch xarm_planner lite6_planner_fake.launch.py [add_gripper:=true]
    # 【real Lite6】launch xarm_planner_node
    $ ros2 launch xarm_planner lite6_planner_realmove.launch.py robot_ip:=192.168.1.117 [add_gripper:=true]

    # 【simulated UFACTORY850】launch xarm_planner_node
    $ ros2 launch xarm_planner uf850_planner_fake.launch.py [add_gripper:=true]
    # 【real UFACTORY850】launch xarm_planner_node
    $ ros2 launch xarm_planner uf850_planner_realmove.launch.py robot_ip:=192.168.1.181 [add_gripper:=true]

    # 다른 터미널에서 테스트 프로그램을 실행 (control through API, specify 'robot_type' as 'xarm' or 'lite' or 'uf850')
    $ ros2 launch xarm_planner test_xarm_planner_api_joint.launch.py dof:=6 robot_type:=<xarm | lite | uf850>
    $ ros2 launch xarm_planner test_xarm_planner_api_pose.launch.py dof:=6 robot_type:=<xarm | lite | uf850>
    ```

    Below additional tests are just for xArm:
    ```bash
    # run test program（control through service）
    $ ros2 launch xarm_planner test_xarm_planner_client_joint.launch.py dof:=6
    $ ros2 launch xarm_planner test_xarm_planner_client_pose.launch.py dof:=6

    # run test program（control gripper through API）
    $ ros2 launch xarm_planner test_xarm_gripper_planner_api_joint.launch.py dof:=6

    # run test program（control gripper through service）
    $ ros2 launch xarm_planner test_xarm_gripper_planner_client_joint.launch.py dof:=6
    ```


- ### 5.8 xarm_gazebo
    이 package는 Gazebo xArm 시뮬레이션을 지원한다.
    ***Notice:***  
    (1) [gazebo_ros2_control](https://github.com/ros-simulation/gazebo_ros2_control.git) 를 소스에서 설치 및 gazebo_ros2_control의 환경 변수를 설정
    (2) [minic_joint_plugin](https://github.com/roboticsgroup/roboticsgroup_upatras_gazebo_plugins) was developed for ROS1, we have modified a version for ROS2 compatibility and it is already integrated in this package for xArm Gripper simulation.  
    
    - gazebo에서 xarm을 독립적으로 테스팅:
        ```bash
        $ cd ~/dev_ws/
        # For xArm (xarm6 here):
        $ ros2 launch xarm_gazebo xarm6_beside_table_gazebo.launch.py

        # For Lite6:
        $ ros2 launch xarm_gazebo lite6_beside_table_gazebo.launch.py

        # For UFACTORY850:
        $ ros2 launch xarm_gazebo uf850_beside_table_gazebo.launch.py
        ```

    - moveit+gazebo로 시뮬레이션 (xArm controlled by moveit).
        ```bash
        $ cd ~/dev_ws/
        # For xArm (xarm6 here):
        $ ros2 launch xarm_moveit_config xarm6_moveit_gazebo.launch.py

        # For Lite6:
        $ ros2 launch xarm_moveit_config lite6_moveit_gazebo.launch.py

        # For UFACTORY850:
        $ ros2 launch xarm_moveit_config uf850_moveit_gazebo.launch.py
        ```
- ### 5.9 xarm_moveit_servo
    이 package는 joystick으로 xArm을 조정하는 데모를 제공한다.  [moveit_servo](http://moveit2_tutorials.picknik.ai/doc/realtime_servo/realtime_servo_tutorial.html). 
    -  __XBOX360__ joystick:
        - 왼쪽 스틱 :  X 와 Y 방향  
        - 오른쪽 스틱 : ROLL 과 PITCH 조정  
        - 왼쪽과 오른쪽 trigger (LT/RT) : z 방향 
        - 왼쪽과 오른쪽 bumper (LB/RB) : Yaw 조정 
        - D-PAD : joint1 과 joint2 제어
        - 버튼 X 와 B : 마지막 joint 제어 
        - 버튼 Y 와 A : 2번째 마지막 joint 제어  

        ```bash
        $ cd ~/dev_ws/
        # XBOX Wired -> joystick_type=1
        # XBOX Wireless -> joystick_type=2
        # For controlling simulated xArm:
        $ ros2 launch xarm_moveit_servo xarm_moveit_servo_fake.launch.py joystick_type:=1
        # Or controlling simulated Lite6:
        $ ros2 launch xarm_moveit_servo lite6_moveit_servo_fake.launch.py joystick_type:=1
        # Or controlling simulated UFACTORY850:
        $ ros2 launch xarm_moveit_servo uf850_moveit_servo_fake.launch.py joystick_type:=1


        # For controlling real xArm: (use xArm 5 as example)
        $ ros2 launch xarm_moveit_servo xarm_moveit_servo_realmove.launch.py robot_ip:=192.168.1.123 dof:=5 joystick_type:=1
        # Or controlling real Lite6:
        $ ros2 launch xarm_moveit_servo lite6_moveit_servo_realmove.launch.py robot_ip:=192.168.1.123 joystick_type:=1
        # Or controlling real UFACTORY850:
        $ ros2 launch xarm_moveit_servo uf850_moveit_servo_realmove.launch.py robot_ip:=192.168.1.181 joystick_type:=1
        ```
        ```

    - Controlling with __3Dconnexion SpaceMouse Wireless__:
        - 6 DOFs of the mouse are mapped for controlling X/Y/Z/ROLL/PITCH/YAW  
        - Left button clicked for just X/Y/Z adjustment  
        - Right button clicked for just ROLL/PITCH/YAW adjustment  

        ```bash
        $ cd ~/dev_ws/
        # For controlling simulated xArm:
        $ ros2 launch xarm_moveit_servo xarm_moveit_servo_fake.launch.py joystick_type:=3
        # Or controlling simulated Lite6:
        $ ros2 launch xarm_moveit_servo lite6_moveit_servo_fake.launch.py joystick_type:=3
        # Or controlling simulated UFACTORY850:
        $ ros2 launch xarm_moveit_servo uf850_moveit_servo_fake.launch.py joystick_type:=3

        # For controlling real xArm: (use xArm 5 as example)
        $ ros2 launch xarm_moveit_servo xarm_moveit_servo_realmove.launch.py robot_ip:=192.168.1.123 dof:=5 joystick_type:=3
        # Or controlling real Lite6:
        $ ros2 launch xarm_moveit_servo lite6_moveit_servo_realmove.launch.py robot_ip:=192.168.1.123 joystick_type:=3
        # Or controlling real UFACTORY850:
        $ ros2 launch xarm_moveit_servo uf850_moveit_servo_realmove.launch.py robot_ip:=192.168.1.181 joystick_type:=3
        ```
    
    -  __PC keyboard__:
        ```bash
        $ cd ~/dev_ws/
        # 시뮬레이션 xArm 제어하기:
        $ ros2 launch xarm_moveit_servo xarm_moveit_servo_fake.launch.py dof:=6
        # 혹은 시뮬레이션 Lite6 제어하기:
        $ ros2 launch xarm_moveit_servo lite6_moveit_servo_fake.launch.py
        # 시뮬레이션 UFACTORY850 제어하기:
        $ ros2 launch xarm_moveit_servo uf850_moveit_servo_fake.launch.py

        # For controlling real xArm: (use xArm 5 as example)
        $ ros2 launch xarm_moveit_servo xarm_moveit_servo_realmove.launch.py robot_ip:=192.168.1.123 dof:=5
        # Or for controlling real Lite6:
        $ ros2 launch xarm_moveit_servo lite6_moveit_servo_realmove.launch.py robot_ip:=192.168.1.123
        # Or for controlling real UFACTORY850:
        $ ros2 launch xarm_moveit_servo uf850_moveit_servo_realmove.launch.py robot_ip:=192.168.1.181

        # Then in another terminal, run keyboad input node:
        $ ros2 run xarm_moveit_servo xarm_keyboard_input
        ```
        Please note that Moveit Servo may consider the home position as singularity point, then try with joint motion first.  

## 6. 중요 launch 인자
- __robot_ip__,
    xArm의 IP 주소. 실제 HW 제시어 필요.
- __report_type__, default: normal. 
    Data report type, 지원하는 types : normal/rich/dev, 
    각 type 마다 다른 data contents와 frequency를 가진다..
- __dof__, default: 7. 
    robot arm의 DOF. 별도로 지정할 필요는 없음.
    dual arm launch 파일(```dual_``` prefix를 가짐)에 대해서 DOF는 다음을 통해서 지정한다.:
    - __dof_1__
    - __dof_2__
- __velocity_control__, default: false. 
    velocity 인터페이스로 제어 여부 (반대로, position 인터페이스 사용)
- __add_realsense_d435i__, default: false.  
    realsense D435i 카메라 모델 로드 여부.
    dual arm launch 파일( ```dual_``` prefix를 가짐)에 대해서 다음을 통해 지정한다.:
    - __add_realsense_d435i_1__
    - __add_realsense_d435i_2__
- __add_gripper__, default: false. 
    model 내부에 UFACTORY gripper를 포함여부. ```add_vacuum_gripper``` 인자보다 더 높은 우선순위를 가진다.
    For dual arm launch files(with ```dual_``` prefix), it can be specified through:
    - __add_gripper_1__
    - __add_gripper_2__
- __add_bio_gripper__, default: false. 
    Whether to include BIO gripper in the model，it has higher priority than the argument ```add_vacuum_gripper```, ```add_gripper``` must be false in order to set vacuum gripper to be true.
    For dual arm launch files(with ```dual_``` prefix), it can be specified through:
    - __add_bio_gripper_1__
    - __add_bio_gripper_2__
- __add_vacuum_gripper__, default: false. 
    Whether to include UFACTRORY vacuum gripper in the model，```add_gripper``` must be false in order to set vacuum gripper to be true.
    For dual arm launch files(with ```dual_``` prefix), it can be specified through:
    - __add_vacuum_gripper_1__
    - __add_vacuum_gripper_2__
- __add_other_geometry__, default: false. 
    Whether to add other geometric model as end-tool, ```add_gripper``` and ```add_vacuum_gripper``` has to be false in order to set it to be true.
    
    - __geometry_type__, default: box, effective when ```add_other_geometry=true```.  
        geometry type to be added as end-tool，valid types: box/cylinder/sphere/mesh.  
    - __geometry_mass__, unit: kg，default value: 0.1  
        model mass.
    - __geometry_height__, unit: m，default value: 0.1  
        specifying geometry hight，effective when geometry_type=box/cylinder/sphere.  
    - __geometry_radius__, unit: m，default value: 0.1  
        specifying geometry radius, effective when geometry_type=cylinder/sphere.  
    - __geometry_length__, unit: m，default value: 0.1  
        specifying geometry length, effective when geometry_type=box.  
    - __geometry_width__, unit: m，default value: 0.1  
        specifying geometry width,effective when geometry_type=box.  
    - __geometry_mesh_filename__,
        filename of the specified mesh model，effective when geometry_type=mesh.  
        ***This file needs to be put in ```xarm_description/meshes/other/``` folder.*** Such that full directory will not be needed in filename specification.  
    - __geometry_mesh_origin_xyz__, default: "0 0 0"  
    - __geometry_mesh_origin_rpy__, default: "0 0 0"  
        transformation from end-flange coordinate frame to geometry model origin coordinate frame, effective when ```geometry_type=mesh```. Example: geometry_mesh_origin_xyz:='"0.05 0.0 0.0"'.  
    - __geometry_mesh_tcp_xyz__, default: "0 0 0"  
    - __geometry_mesh_tcp_rpy__, default: "0 0 0"  
        transformation from geometry model origin frame to geometry model tip ("Tool Center Point") frame, effective when ```geometry_type=mesh```. Example: geometry_mesh_tcp_rpy:='"0.0 0.0 1.5708"'.  
    - __Example of adding customized end tool (Cylinder):__  

        ```bash
        $ ros2 launch xarm_gazebo xarm6_beside_table_gazebo.launch.py add_other_geometry:=true geometry_type:=cylinder geometry_height:=0.075 geometry_radius:=0.045
        ```

    For dual arm launch files(with ```dual_``` prefix), here are the total arguments that can be configured:  
    - __add_other_geometry_1__
    - __add_other_geometry_2__
    - __geometry_type_1__
    - __geometry_type_2__
    - __geometry_mass_1__
    - __geometry_mass_2__
    - __geometry_height_1__
    - __geometry_height_2__
    - __geometry_radius_1__,
    - __geometry_radius_2__,
    - __geometry_length_1__
    - __geometry_length_2__
    - __geometry_width_1__
    - __geometry_width_2__
    - __geometry_mesh_filename_1__
    - __geometry_mesh_filename_2__
    - __geometry_mesh_origin_xyz_1__
    - __geometry_mesh_origin_xyz_2__
    - __geometry_mesh_origin_rpy_1__ 
    - __geometry_mesh_origin_rpy_2__ 
    - __geometry_mesh_tcp_xyz_1__
    - __geometry_mesh_tcp_xyz_2__
    - __geometry_mesh_tcp_rpy_1__
    - __geometry_mesh_tcp_rpy_2__
- __kinematics_suffix__: joint Kinematics 파라미터 파일 suffix를 지정
    - Kinematics 파리미터 파일 생성: 
      ```bash
      cd src/xarm_ros/xarm_description/config/kinematics
      python gen_kinematics_params.py {robot_ip} {kinematics_suffix}

      # Note
      # 1. robot_ip 는 robot arm의 IP 주소. 실제 파라미터를 얻으려면 robot arm에 연결해야한다.
      # 2. kinematics_suffix represents the suffix of the generated parameter file. If successful, the configuration file will be generated in the xarm_description/config/kinematics/user directory. If kinematics_suffix is AAA, then the corresponding file name is as follows
      #   xarm5: xarm_description/config/kinematics/user/xarm5_kinematics_AAA.yaml
      #   xarm6: xarm_description/config/kinematics/user/xarm6_kinematics_AAA.yaml
      #   xarm7: xarm_description/config/kinematics/user/xarm7_kinematics_AAA.yaml
      #   lite6: xarm_description/config/kinematics/user/lite6_kinematics_AAA.yaml
      #   uf850: xarm_description/config/kinematics/user/uf850_kinematics_AAA.yaml
      ```
    - Kinematics parameter 파일 사용: launch 파일을 시작할때 이 parameter를 지정
      - Note that before specifying this parameter, make sure that the corresponding configuration file exists. If it does not exist, you need to connect the robot arm through a script to generate it.