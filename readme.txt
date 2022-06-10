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
	phx-launch -d 8 phoenix_launch experiment.xlaunch name:=romulus > 202202171134test.out
	rosnode info /romulus/object_detector 

install prep:
	cd phoenix/phoenix-r1/
	./build-tools/install_ros_deps.sh

normal workflow:
	ssh robot@romulus to jackel, passwd BotLangBot!@#$
	then run the phxlaunch
	

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
