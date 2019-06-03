# Tutorials

## _Tutorial 1:_ Getting the car running

1. Turn the engine on by turning the key completely to the right
2. On the right side of the trunk, rotate the power button to the left
3. Press the power button of the IPC until you hear a sound
4. Turn the iPad on
5. Wait a few minutes until you hear a short melody coming from the IPC, indicating that the boot sequence is complete
6. Open the DreamView app on the iPad to control Apollo
7. On the top right of the screen, select either  "Pandora" or "RTK Record/Replay'' then "Kia Niro" and "Gloshaugen"
8. Press the Setup button in the bottom left part of the screen
9. If you are in RTK Record/Replay mode, you can then press the RTK Record Start/Stop buttons to record a trajectory. Then, RTK Replay Start to publish this trajectory as planning. Then Start Auto to let the car drive along this trajectory

## _Tutorial 2:_ Acquiring sensors data

### Acquiring Pandora data from a laptop

For convenience, we recommend using an Ubuntu laptop to acquire Pandora data. On the laptop, .

Configure the Ethernet interface to be on the Pandora’s local network. You can use network-manager to setup the manual IPv4 address 192.168.20.100.

Clone, compile and run the Pandora\_ros ROS node.

```text
mkdir -p rosworkspace/src
cd rosworkspace/src
git clone --recursive https://github.com/HesaiTechnology/Pandora_ros
cd ..
catkin_make_isolated
source devel/setup.sh
roslaunch pandora pandora_driver.launch
```

You can now also run the _rviz_ command and add PointCloud and Image viewers corresponding to the topics emitted by pandora\_driver to visualize the data in real-time.

To record data, open a new terminal and type:

```bash
rosbag record -a
```

When you are done, just use Ctrl-C and wait for it to finish filling the file in.

_Note:_ The same principle can be used to acquire data from the VLP16, by compiling the ROS node instead of Pandora\_ros.

_Note:_ The same principle can be used to acquire data on the IPC \(eg: LIDAR+GPS+Perception output etc...\), by running _rosbag record_ from inside the Apollo Docker container or by starting the _Record bag_ module from DreamView, while the other modules are running. The Record bag module from DreamView will record all topics on an external hard disk if connected or only the important topics on the IPC if no disk is connected.

## _Tutorial 3:_ Reinstalling the IPC

1. Download
2. Flash it on a USB stick using _dd_ or _unetbootin_
3. Reboot the IPC and while it boots, press F12
4. Select the USB drive “Generic Flash Disk”
5. Set the language to English
6. Set the timezone to Other, Europe, Norway
7. Set the locale to United States en\_US.UTF-8
8. Don’t detect layout, select Norwegian
9. Set the disk to p5p1
10. Set the hostname to kia-niro
11. Set the full name to NAP Lab
12. Set the username to naplab
13. Set the password to 123456
14. Accept the weak password
15. Do not accept disk encryption
16. Accept the Europe/Oslo timezone
17. Unmount /dev/sda
18. Choose Guided - use entire disk
19. Choose /dev/nvme0n1
20. Accept to write changes to disk
21. Leave the package manager proxy blank
22. Select no automatic updates
23. Select OpenSSH server only
24. Install the GRUB boot loader to the master boot record
25. The system clock is set to UTC
26. Continue and remove the USB stick
27. Reboot the machine, connect and run the following commands:

```bash
sudo apt update
sudo apt upgrade
sudo apt install git make gcc libssl-dev beep
sudo curl -sSL https://get.docker.com/ | sh
git clone https://github.com/FlorentRevest/apollo
git clone https://github.com/FlorentRevest/apollo-kernel
cd apollo-kernel/linux
./build.sh rt
cd install/rt/
tar xvf install.tgz
cd install
sudo ./install_kernel.sh
sudo vi /etc/network/interfaces
```

Edit the _/etc/network/interfaces_ file to have this content:

```bash
auto lo
iface lo inet loopback

allow-hotplug p5p1
iface p5p1 inet dhcp
iface p5p1 inet6 auto

allow-hotplug p6p1
iface p6p1 inet static
    address 192.168.20.100
    netmask 255.255.255.0

sudo usermod -aG docker naplab
sudo sed -i "s@GRUB_DEFAULT=0@GRUB_DEFAULT=saved@" /etc/default/grub
sudo sed -i "s@GRUB_TIMEOUT=10@GRUB_TIMEOUT=1@" /etc/default/grub
sudo grub-set-default 1
sudo update-grub
reboot
```

Wait for the IPC to reboot...

```bash
cd apollo-kernel/linux/
sudo ./install-nvidia.sh
reboot
```

Wait for the IPC to reboot again...

```bash
cd apollo
sed -i 's@<username>@fmrevest@' modules/calibration/data/kia_niro/gnss_params/gnss_conf.pb.txt
sed -i 's@<password>@4rS42sJh@' modules/calibration/data/kia_niro/gnss_params/gnss_conf.pb.txt
sed -i 's@api/js@api/js?key=AIzaSyDii-Y-bxQXWeUdnyZnBjszQS5rFym_HXQ@' modules/dreamview/frontend/src/store/config/parameters.yml
sudo sed -i "/snd_pcsp/d" /etc/modprobe.d/blacklist.conf
sudo sed -i "/pcspkr/d" /etc/modprobe.d/blacklist.conf
sudo sh -c 'echo "UUID=562032004DA2CCDB /media/external_hdd ntfs-3g  auto,nobootwait,users,uid=1000,gid=100,dmask=027,fmask=137,utf8 0 0" > /etc/fstab'
docker/scripts/dev_start.sh
```

Edit _/etc/rc.local_ using nano or vi and add before before “_exit 0_”:

```bash
sudo -H -u naplab /home/naplab/apollo/docker/scripts/dev_start.sh
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
sudo ip link set can1 type can bitrate 500000
sudo ip link set up can1
beep -f165.4064 -l100 -n -f130.813 -l100 -n -f261.626 -l100 -n -f523.251 -l100 -n -f1046.50 -l100 -n -f2093.00 -l100 -n -f4186.01 -l100

docker/scripts/dev_into.sh
./apollo.sh build_opt_gpu
```

