hathorvision network is configured using netplan for assign fixed ip or not

when running on a different machine, the network config need to be adjusted
e.g., when running on jackle stock computer, need to add enp4s0 interface
and add it to br0 to be able to talk to arl robot network.

see:
        /etc/netplan/###.yaml

after change, sudo netplan apply

networkctl 
IDX LINK    TYPE     OPERATIONAL SETUP     
  1 lo      loopback carrier     unmanaged
  2 enp4s0  ether    degraded    unmanaged 
  3 eno1    ether    no-carrier  unmanaged 
  4 wlp3s0  wlan     no-carrier  unmanaged 
  5 br0     bridge   no-carrier  configured
  6 br1     bridge   routable    configure

/etc/netplan/###.yaml
  bridges:
    br0:
      macaddress: 78:d0:04:28:63:c7
      addresses: [192.168.91.181/23]
      gateway4: 192.168.90.1
      interfaces: [enp0s31f6, enp10s0, enp9s0,enp4s0]
      nameservers:
        addresses: [131.218.141.80, 131.218.141.21]

------------------------- hathorvision, baal, etc images ------

hathorvision:
	sda1: hathorvision, full desktop, netplan
	sda3: baal
	grub 
baal:
	*sda1: baal, headless, netplan
	sda8: baal backup, new UUID
	sda6: arldell
	grub not really working, 
hathor:
	sda1: hathor, headless, netplan
