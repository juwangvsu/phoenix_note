-------sticky 6/9/22 -------------------
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
>xhost +si:localuser:root
catkin-docker run
	inside docker:
		catkin build #this rebuild everyting, take long time
			the output folder: docker-build, this is 
			mounted (-v) at when call "docker run", actual space
			is somewhere docker partition (dockerimg in my case).
			it is not visible in host machine.
		catkin build hardware_launch --no-deps #this rebuild part of it, fast

switching branch build:
	catkin clean
	catkin build
	might need to exit docker, checkout the branch, then build
	cdea_arl_objectmapper: building...
		ocrvio build fail, missing sophus pkg
			debugging in docker:
				source /opt/ros/noetic/setup.bash
				sudo apt update
				sudo apt install ros-noetic-sophus
				build work.
			or use a different docker image	
	master: build ok
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

------------ 6/9/22 hardware info --------------------------

	jackel ship with a mini-pc, intel i5, 8gb, geforce 1050
	John R. teams use an additional nuvo 7160 computer, which
	is a intel nuc i7 up to 64 gb with gpu slot 
	
------------ 6/9/22 hardware issues --------------------------

	romulus gpu faulty?
		with gpu, take longer to boot, vga no output. nvidia-smi
			fail to detect. or could be the driver problem?
			current driver is ~/NVIDIA-Linux-x86_64-495.46.run
			 new enough. history shows that this one is installed.

		without gpu, boot up quick
