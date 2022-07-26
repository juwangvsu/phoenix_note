------7/26/22 infosploration ddf code reading --------------
nodes @ robot:(thoth)
	/thoth/omnigraph
		posegraph, ddf, loop closure
	/thoth/map_merger
		simple map merge, use the submap pose to merge map
	/thoth/rtm_robots
		remote topic manager with other robots
	/thoth/rtm
		remote topic manager with control station
	
------7/25/22 infosploration test --------------
changed up robot-robot-control topic management
	reliable_udp_comms rtm is more powerful thought. a single rtm at control can manage all robots' connection.
previous launch file use many rtm pairs between robot and control, which is awkward and under-utilizing rtm

sobek:
        phxlaunch infosploration hardware_experiment.xlaunch name:=sobek robot_id:=1 num_robots:=2 partition_mode:=3 peer_robot1:=sobek peer_robot2:=thoth name2:=thoth use_original_comms:=true

control:
        phxlaunch infosploration control_station.xlaunch num_robots:=2 use_original_comms:=true

-------------------7/25/22 map --------
control station:
	/sobek/point_cloud_cache/renderers/full_map_global 
		nav_msgs/OccupancyGrid
		publisher: /sobek/rtm_sobek 

-------------------7/22/22 control_stataion infosploration --------
(0) cmdline:

phxlaunch infosploration control_station.xlaunch num_robots:=5
ROS_IP=192.168.91.121
ROS_MASTER_URI=http://localhost:11311
 -- there should be no ROS_HOSTNAME
    this is very important. unset ROS_HOSTNAME if necessary

to minimize impact on other robot info in this launch file, 
baal can only be added as name5, otherwise the ip address of others must be changed accordingly

use_service_discovery are true on laptop and robots. so their rtm (remote topic manager) find each other automatically

(1) control_station.xlaunch
"use_service_discovery" default="true" 
	this set "use_service_discovery" to default="true"

(2) unreliable_comms_laptop_include.launch
	add baal as name5/host5...

    <arg name="name1" default="sobek"/>
    <arg name="name2" default="hathor"/>
    <arg name="name3" default="ptah"/>
    <arg name="name4" default="apis"/>
    <arg name="name5" default="baal"/>
    <arg name="host1" default="192.168.90.183"/>
    <arg name="host2" default="192.168.90.181"/>
    <arg name="host3" default="192.168.90.184"/>
    <arg name="host4" default="192.168.90.187"/>
    <arg name="host5" default="192.168.90.168"/>
    <arg name="robot_port1" default="11411"/>
    <arg name="robot_port2" default="11413"/>
    <arg name="robot_port3" default="11415"/>
    <arg name="robot_port4" default="11417"/>
    <arg name="robot_port5" default="11419"/>

    <arg name="laptop_port1" default="11412"/>
    <arg name="laptop_port2" default="11414"/>
    <arg name="laptop_port3" default="11416"/>
    <arg name="laptop_port4" default="11418"/>
    <arg name="laptop_port5" default="11420"/>

...

    <include if="$(eval num_robots > 4)" ns="$(arg name5)" file="$(find infosploration)/launch/unreliable_comms_laptop_one.launch">
        <arg name="name"  value="$(arg name5)"/>
        <arg name="remote_host" value="$(arg host5)"/>
        <arg name="robot_port" value="$(arg robot_port5)"/>
        <arg name="laptop_port" value="$(arg laptop_port5)"/>
    </include>

control station nodes:
/joy_relay
/rosout
/rviz
/sobek/remote/teleop_twist_joy
/sobek/remote_host_discovery
/sobek/rtm_sobek
/thoth/remote_host_discovery
/thoth/rtm_thoth

--------------- robot launch file infosplation -------------
hardware_experiment.xlaunch
	phoenix_launch)/launch/robot.xlaunch
	infosploration)/launch/unreliable_comms_robot_direct.launcha
		robot to robot direct communications
	infosploration)/launch/unreliable_comms_robot_2.launch
		robot to base station pair-wise
         	communications over relieable UDP
	infosploration)/launch/mr_infosploration_include.launch
	unreliable_comms_launch)/launch/ddf.xlaunch


unreliable_comms_robot_direct.launch: (robot-robot talk)
	name=robot_remote_host_discovery pkg="reliable_udp_comms" type="remote_host_discovery.py group_name" value="MRINFO_robot"
		this find remote host
	name=rtm_robots pkg="reliable_udp_comms" type="remote_topic_manager 
		this run rtm

unreliable_comms_robot_2.launch
	name="remote_host_discovery" pkg="reliable_udp_comms" type="remote_host_discovery.py" "group_name" value="RC_$(arg name)"/
	name="rtm" pkg="reliable_udp_comms" type="remote_topic_manager" 
	
