-------sticky 6/9/22 -------------------
BIPB passwd: $1RDRL-CII-B2$
baal passwd: BotLangBot!@#$

run jackel stack:
	cd phoenix/phoenix-r1/
	source install/setup.bash
	phxlaunch phoenix_launch experiment.xlaunch name:=romulus
	xbox controller driving:
		right upper trigger + left joystick

sudo bash gimmeinternet.sh 

Rebuild:
	cd phoenix/phoenix-r1/
	source install/setup.bash
	catkin build phoenix_launch --no-deps

other test:
	roslaunch hardware_launch microstrain.launch port:="/dev/ttyACM1" sensor_name:=imu name:=romulus
	roslaunch microstrain_inertial_driver microstrain.launch port:="/dev/imu"
	phxlaunch -d 8 phoenix_launch experiment.xlaunch name:=romulus > 202202171134test.out
	rosnode info /romulus/object_detector 

install prep:
	cd phoenix/phoenix-r1/
	./build-tools/install_ros_deps.sh

normal workflow:
	ssh robot@romulus to jackel, passwd BotLangBot!@#$
	then run the phxlaunch

launch seq/pkgs:
	launch-files/phoenix_launch ---- main launch files
		example/experiment.xlaunch  ---(1)
		launch/robot.xlaunch----------(2)
		  launch/hardware.xlaunch   --- (3)
		  launch/control.xlaunch    --- (4)
		  launch/stack.xlaunch	 --- (6)

	launch-files/hardware_launch --- hardware launch config filesa
		launch/platforms/husky.launch  (3.1)
		launch/platforms/realsense.launch  (3.2)
		launch/platforms/laser.launch  (3.3)
						(...)
		launch/description.launch (5)
		config/husky.yaml ------- 

	drivers/platforms/husky/husky_control
		launch/control.launch	----- (4.1)
		launch/teleop.launch    ----- (4.2)

	stack.xlaunch: (6.1-4)
		
  <rosarg arg="functions">
    # === Perception functions ===
    object_pipeline: $(find perception_launch)/launch/object_pipeline.xlaunch
    classification_pipeline: $(find perception_launch)/launch/classification_pipeline.launch
    # mover_detection
    # mover_prediction

    # === Intelligence functions ===
    intelligence: $(find intelligence_launch)/launch/intelligence.launch
    navigation_behaviors: $(find intelligence_launch)/launch/navigation_behaviors.launch

    # === Mapping functions ===
    odometry: $(find estimation_launch)/launch/odometry.launch
    slam: $(find mapping_launch)/launch/mapping.xlaunch
    terrain_projection: $(find perception_launch)/launch/terrain_projection.launch

    # === Navigation functions ===
    navigation: $(find navigation_launch)/launch/navigation.launch
    ioc_traversal: $(find navigation_launch)/launch/ioc_traversal.launch
    # social navigation
  </rosarg>

----7/18/22  navigation config files  -------a
	launch-files/aimm_mobility_experiments/launch
		hardware_experiment.xlaunch
			tag <experiment_params default="experiment_baseline.yaml">
			experiment_baseline.yaml (or customize it)
				navigation related parameters

baal nav test:
	bug fixed:
		global planner need costmap resolution and lidar resolution to be the same. costmap resolution is 0.1. velyndye was set to 0.2. this cause planner to crash.
		fix is set center_lidar_keyframe_cloud_to_grid/resolution : 0.1

----7/13/22  unity demo run  -------
optin install:
	pyenv shell mini...
	cd main folder	
	vcs import -w 1 --repos --input $(catkin locate phoenix_unity_optin)/unity_packages.yaml $(catkin locate phoenix_unity_optin)/..
		this will download unity simulator to the main folder

	run docker,
		source ~/phoenix-r1/docker-build/install/setup.bash 
		phxlaunch phoenix_unity_launch experiment.xlaunch
			this will launch both rviz and unity
	note on unity simulator:
		the object detection is the ground truth from the unity command, so
		it did not really run the object detection node.
	
----7/10/22  demo run hathor -------
robot passwd: 1Amsrl-ci-cB2

robot side: (main computer)
	headless
	baal: cdea_arl, headless, 6/2022, jackel (stock)
	hathor: hotfix_tfprefix, headless, 6/2022, jackel (stock), clone-> hathor(c)
	anubis: warty_test, with gdm, 1/2022, husky (nuvc), @baal
		
	ssh robot@hathor
	phxlaunch aimm_mobility_experiments hardware_experiment.xlaunch name:=hathor attempt_match:=false     

control station side:
	
	xhost +si:localuser:root
	catkin-docker run
		(*) source docker-build/install/setup.bash
		export ROS_MASTER_URI=http://hathor:11311
		export ROS_IP=192.168.91.121
	phxlaunch aimm_mobility_experiments control_station.xlaunch name:=hathor

control station join a running docker container: (e.g., want to run to control station to deal with two robot)
	docker exec --privileged -e DISPLAY=${DISPLAY} -it ff84e5c1ed09 bash
		then same as (*)

vision computer:
	nuvo machine. 
	seems a run a desktop with gdm
	hathorvision: cdea_arl, wih gdm, jackel (nuvc), clone->hathorvision(c)
	use ros feature machine to run nodes at a different computer

jackel two computer setup:
	drive chain -> main. alt conn: micro usb on the jackel drive chain port
	main<-> vision: ethernet
	lidar -> main.ethernet
	bullet-> vision.ethernet

	jackel main computer:
		lidar, lidar pipeline, ommnimapper, navigation stack
	jackel vision computer:
		run perception_launch/launch/object_pipeline/object_detection.launch etc

most freq:
	vi control_station.xlaunch control.xlaunch robot.xlaunch

----7/13/22 testing some perception pkgs ---------------------
https://gitlab.sitcore.net/aimm/phoenix-r1/-/blob/master/src/perception/models/README.md
perception_models optin:
	this download model parameters, such as darknet
roslaunch darknet_phoenix darknet_phoenix.launch
	c++ code
	it first ask yolo model to be downloaded. did that with perception optin readme
	then it runs but error on a runtime cuda error
roslaunch rcta_object_pose_detection rcta_object_pose_detection.launch
	seems working, pytorch code

-----6/21/22 build rerun -----------------------
docker:
	sudo mount -o remount,suid /media/student/dockerimg/
	clone repo to elite for build
	arldell csm_itmp branch build not work. rerun build
	with master branch ok after reclone.
git lfs:
	curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo os=ubuntu dist=`lsb_release -sc` bash
	sudo apt install git-lfs
catkin-docker:
	cd ~
	git clone https://gitlab.sitcore.net/aimm/catkin-docker.git
	echo 'export PATH=~/catkin-docker:$PATH' >> ~/.bashrc
	cp ~/catkin-docker/home_example.catkin_docker ~/.catkin_docker

git clone https://gitlab.sitcore.net/aimm/phoenix-r1.git
git fetch #this pull all new branches? otherwise some branch don't show
git checkout xxxx # e.g., cdea_arl_objectmapper
cd phoenix-r1

miniconda-latest
python3 build-tools/catkin_config/catkin_profile_build
echo "active: phoenix" > .catkin_tools/profiles/profiles.yaml
docker login registry.gitlab.sitcore.net:443
docker pull registry.gitlab.sitcore.net:443/aimm/phoenix-r1/noetic/devel:master

(docker run):
(1) xhost +si:localuser:root
(2) catkin-docker run
	inside docker:
		catkin build #this rebuild everyting, take long time
			the output folder: docker-build, this is 
			mounted (-v) at when call "docker run", actual space
			is somewhere docker partition (dockerimg in my case).
			it is not visible in host machine.
		catkin build hardware_launch --no-deps #this rebuild part of it, fast

switching branch build:
	possible problem: when build multiple branch, they might confuse each other since
		the build process use the same pat of docker storage to mount as 
		"docker-build", if the same docker image is used to run "catkin build".
		 so if we build master branch, then build cdea_arl... 
		the later might fail due to the existing master build
	arldell:
		/media/student/data6/phoenix-cdea_arl_objectmapper/ master 7/13/22 build ok
		/media/student/data6/phoenix-master cdea_arl_objectmapper 6/23/22 ok 7/13/22 ok see (**)  
		/media/student/data6/phoenix-r1-0  csm_itmp--- mixedup
		/media/student/data6/phoenix-r1  master , build cdea_arl_objectmapper nogood list index out of range
		/media/student/data6/phoenix-r1.zip
	lenova1:/media/student/data5/phoenix-2/phoenix-r1	cdea_arl_objectmapper
		lenova1:/media/student/data5/phoenix-r1	csm_itmp, amino_ros fail

	catkin clean
	catkin build
	might need to exit docker, checkout the branch, then build
	(**) cdea_arl_objectmapper: build ok subject to do this everything to build
		ocrvio build fail, missing sophus pkg
			debugging in docker:
				source /opt/ros/noetic/setup.bash
				sudo apt update
				sudo apt install ros-noetic-sophus
				build work.
			or use a different docker image	
	csm_ltmp: building...
		customized docker file Dockerfile_amino
		in docker:
			git clone amino, make, make install..
		catkin build
			catkin build amino_ros
			 still fail:
			/amino-1.0/amino/rx/scene_gl.h:41:10: fatal error: SDL_opengl.h: No such file or directory
			

	master: build ok

mri-exploration-cdea:
	docker build:
		need sophus install 
		
-------------------------------------------------

testing the system:
# Outside docker
cd phoenix-r1
mkdir bags
(cd bags && curl -O https://arl-aimm-data-public.s3.amazonaws.com/rcta_t2_marketplace_th05_short.bag)
cd ..

# Enter a catkin-docker container
catkin-docker run

# Set up the phxlaunch command
source docker-build/install/setup.bash

# Generate a fully resolved launch file of the given bag
rosrun phoenix_bag_launch generate_bag_args $(catkin locate)/bags/rcta_t2_marketplace_th05_short.bag -o mybag.launch
roslaunch mybag.launch

# When desired, exit docker
exit

--------6/19/22 mybag.launch ------------------------------
from the main readme. important nodes:
/samoyed/executor
/samoyed/global_costmap
/samoyed/global_planner
/samoyed/map_c
/samoyed/map_d
/samoyed/nlu
/samoyed/omnigraph
/samoyed/point_cloud_pipeline/center_lidar_centroids
	and 10 other nodes 

topics:
	/pose_graph
/samoyed/assembled_cloud
/samoyed/assembled_cloud_lowres

bags/...bag topics:
/samoyed/fast_assembled_cloud_lowres  pointcloud2, rviz ok

/samoyed/full_cloud
full node list:
	nodelist.txt 	
full topic list:
	topiclist.txt 	

------------ 6/9/22 hardware info/ network config with bullet --------------------------

	jackel ship with a mini-pc, intel i5, 8gb, 
	gpu: geforce 1050 ti 4gb, 750, 730 from 1-4gb
	John R. teams use an additional nuvo 7160 computer, which
	is a intel nuc i7 up to 64 gb with gpu slot 

network config:
baal network interface:
eno1:	192.168.10.50	lidar
enp4s0:	192.168.90.168	wifi via bullet
bullet:
	bullet need to be connected to a ethernet port. The computer ethernet port should
		be configured static ip (192.168.90.xxx). for two computers to comm, you 
		need to know what ip you configured for it.

the system check for both network interfaces at boot. if any of them not ready, booting will wait for 2 minutes
before proceed.

netplan: (baal)
	/etc/netplan/01_default.yaml
  2   ethernets:                                                                                                  
  3     enp4s0:                                                                                                   
  4       addresses: [192.168.90.168/23]                                                                          
  5       gateway4: 192.168.90.1                                                                                  
  6     enp3s0:                                                                                                   
  7       addresses: [192.168.90.168/23]                                                                          
  8       gateway4: 192.168.90.1                                                                                  
  9     eno1:                                                                                                     
 10       addresses: [192.168.10.50/24]                                                                           
 11       #enx8cae4ce95092:                                                                                       
 12       #  addresses: [10.44.0.1/24]   

--- Bullet configure:
	connect to bullet wifi
	https://192.168.172.1/#network
	login: ubnt
	passwd: 1Amxxxx
	setting: see bullet-conf.zip

	
------------ 6/9/22 hardware issues --------------------------

	romulus gpu faulty?
		with gpu, take longer to boot, vga no output. nvidia-smi
			fail to detect. or could be the driver problem?
			current driver is ~/NVIDIA-Linux-x86_64-495.46.run
			 new enough. history shows that this one is installed.

		without gpu, boot up quick
	baal, gpu display faulty, 4gb, ub20, headless (no x server)? startup long waiting for a job?
if boot stuck, check systemd service    
-----------------baal booting hang on network service ---------                                                                                         
  2                                                                                                               
  3 sudo systemctl enable systemd-networkd-wait-online.service                                                    
  4   enable a service                                                                                            
  5 systemctl  list-units --type=servicea                                                                         
  6   list service showing status                                                                                 
  7 sudo systemctl disable systemd-networkd-wait-online.service                                                   
  8   disable service                                                                                             
  9 systemd-analyze blamea                                                                                        
 10   show which service take how long time                                                                       
 11 sudo systemctl disable ptpd.service                                                                           
 12                                                                                                               
 13                                                                                                               
 14 ---- issue resolved ----------------                                                                          
 15 it turn out that the networkd-wait-online.service is required by the following services (they has a wanted and after tag in scrip
 16 so when starting these service, it will automatically trying to run networkd-wait-online service.             
 17 the one in charge of login tty is open-iscsi.service. so if we want a quick login  screen, we can just comment off the Wants and
 18 in script open-iscsi.service:                                                                                 
 19   vi /etc/systemd/system/sysinit.target.wants/open-iscsi.service                                              
 20 This service will proceed as long as the network interface have carrie status. so a quick hack would be plug  
 21 the network interface to any live ethernet port.                                                              
 22 normal boot for ball is about 48 secs.                                                                                                               
 21                                                                                                               
 22 systemd-networkd-wait-online.service                                                                          
 23 ● └─network-online.target                                                                                     
 24 ●   ├─clamav-freshclam.service                                                                                
 25 ●   ├─docker.service                                                                                          
 26 ●   ├─iscsid.service                                                                                          
 27 ●   ├─open-iscsi.service                                                                                      
 28 ●   └─ptpd.service          

----------------trouble shooting FAQ ------------

xbox joystick not working:
	no ros topic published /.../joy	
	lsusb show device detected, no /dev/input/js0. 
	this is misteriously solved. maybe wire too crowded and usb not
	well.

	js0 won't show until xbox controller synced to receiver.
	launch file is pretty robust
	
hardware_launch at robot complain about device polling fail:
	check lidar power plug, lidar ethernet connector

bullet network interface fragile:
	wiggleing wires in robot cause network disconnect. possibly reason
	poor ethernet quality/design.

Bullet performance: laptop <-> robot @ 202 4 doors apart 1.8 MB/S, 
		same room up to 11 MB/S

------------ tips -------------------------
~/phoenix-r1/src/launch-files/aimm_mobility_experiments$ catkin build --this --no-deps
	build this pkg

rosparam dump | grep 0\.2 
grep -rI Primitive\ resolution\ 
	grep over all files containing "Primitive resolution"
