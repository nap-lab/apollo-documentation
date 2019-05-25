# Software Platform

## Pwrpak7 configuration

The INS module has been configured once via a remote terminal in its web interface using the following series of initialisation commands. The meaning of each command is documented on Novatel’s website. The most important line is “SETINSTRANSLATION ANT1” which defines the distance between the GNSS Antenna and the IMU. This distance has been found using a method described in the Calibration section of this document.

```bash
WIFICONFIG OFF
UNLOGALL THISPORT
INSCOMMAND ENABLE
SETIMUORIENTATION 5
ALIGNMENTMODE AUTOMATIC
VEHICLEBODYROTATION 0 0 0
SERIALCONFIG COM1 9600 N 8 1 N OFF
SERIALCONFIG COM2 9600 N 8 1 N OFF
SERIALPROTOCOL COM2 RS232
INTERFACEMODE COM1 NOVATEL NOVATEL ON
INTERFACEMODE COM2 NOVATEL NOVATEL ON
INTERFACEMODE USB2 RTCMV3 NONE OFF
PPSCONTROL ENABLE POSITIVE 1.0 10000
MARKCONTROL MARK1 ENABLE POSITIVE
EVENTINCONTROL MARK1 ENABLE POSITIVE 0 2
RTKSOURCE AUTO ANY
PSRDIFFSOURCE AUTO ANY
SETINSTRANSLATION ANT1 0.344 0.301 0.9246 0.003 0.003 0.003
SETINSTRANSLATION USER 0.00 0.00 0.00
EVENTOUTCONTROL MARK2 ENABLE POSITIVE 999999990 10
EVENTOUTCONTROL MARK1 ENABLE POSITIVE 500000000 500000000

LOG COM2 GPRMC ONTIME 1.0 0.25
LOG USB1 GPGGA ONTIME 1.0
LOG USB1 BESTGNSSPOSB ONTIME 1
LOG USB1 BESTGNSSVELB ONTIME 1
LOG USB1 BESTPOSB ONTIME 1
LOG USB1 INSPVAXB ONTIME 1
LOG USB1 INSPVASB ONTIME 0.01
LOG USB1 CORRIMUDATASB ONTIME 0.01
LOG USB1 RAWIMUSXB ONNEW 0 0
LOG USB1 MARK1PVAB ONNEW

LOG USB1 RANGEB ONTIME 1
LOG USB1 BDSEPHEMERISB ONTIME 15
LOG USB1 GPSEPHEMB ONTIME 15
LOG USB1 GLOEPHEMERISB ONTIME 15

LOG USB1 INSCONFIGB ONCE
LOG USB1 VEHICLEBODYROTATIONB ONCHANGED

SAVECONFIG
```

## Operating System

The IPC was installed with a server version of Ubuntu 14.04. In order to access the NVIDIA GPU, the NVIDIA 375.39 is installed. The OS is also installed with an OpenSSH server so that one can easily connect to the IPC using ssh naplab@192.168.1.50.

The OS is kept minimal to keep the maximum system ressources for the autonomous driving stack.

In replacement of the Ubuntu 14.04 Linux kernel, Baidu provides a custom “4.4.32-apollo” kernel patched with PREEMPT-RT to provide real time capabilities. The kernel also contains a patch that enables the usage of NVIDIA drivers in the context of a RT kernel. In addition to those patches, it was also necessary to backport a kvaser usb module from a more recent kernel so that the OS support the CAN USB module used by the DriveKit over. Source code for this module is to be found in the “apollo-kernel” directory.

The network is configured so that the leftmost ethernet port uses the 192.168.20.100 IP address, on the HESAI Pandora local network and the rightmost ethernet port uses DHCP but is always assigned the IP address 192.168.1.50.

## Tools

A variety of tools are installed on the IPC to make the car development easier. The package “can-utils” provides tools such as candump which can be used to inspect the status of CAN buses. For instance with “candump can0” one can view frames published on the Vehicle CAN Bus.

An installation of oscc-joystick-commander is also provided, this is a tool that uses the OSCC API to control the car, it also uses the Linux evdev input subsystem to access a joystick \(such as an XBox 360 controller\) and allow an operator to control the car using a joystick.

## Autonomous driving stack

### Introduction to Baidu Apollo

The autonomous driving stack running in our KIA Niro is based on an open-source framework to speed up development and reach state-of-the-art features in a short amount of time. Two major open-source autonomous driving software exist today: Baidu Apollo and Tier-IV Autoware. Baidu Apollo benefits from the best industrial support with many automotive companies already manufacturing vehicles running Apollo. Apollo was then chosen as the autonomous driving stack of our car.

At the time of writing, the latest version of Apollo is 3.5. However, the version 3.0 of Apollo was chosen to be integrated to the car because 3.5 requires a new set of sensors namely: a 128-beam LIDAR which is quite expensive, multiple 16 beams LIDARs, additional RADARs and an "Apollo Sensor Unit" which is only sold by Baidu to selected partners.

Apollo being an open-source framework, one can find a lot of information about it on the Web. The main place to look at is the official documentation on GitHub. It is also recommended to read GitHub issues which contain a lot of non-trivial information and usecases. A couple of website gather information about the framework \href{[https://github.com/kk2491/Apollo\_Auto}{in](https://github.com/kk2491/Apollo_Auto}{in) english} but the best references are in chinese: \href{[https://github.com/YannZyl/Apollo-Note}{Apollo-Note}](https://github.com/YannZyl/Apollo-Note}{Apollo-Note}) and \href{[http://www.fzb.me/apollo/}{fzb.me}](http://www.fzb.me/apollo/}{fzb.me}). Apollo has \href{[https://talk.apolloauto.io/}{a](https://talk.apolloauto.io/}{a) forum} which is not very active, it is recommended to ask questions as GitHub issues. A couple of instant-messaging options are available, mainly the apollodev wechat channel where discussions happen in chinese or the \#apollo IRC channel on freenode which is unfortunately not very active. One can also find classes documentation on doxygen.

Apollo being an open-source framework, one can find a lot of information about it on the Web. The main place to look at is the official documentation on GitHub. It is also recommended to read GitHub issues which contain a lot of non-trivial information and usecases. A couple of website gather information about the framework [in english](https://github.com/kk2491/Apollo_Auto) but the best references are in chinese: [Apollo-Note](https://github.com/YannZyl/Apollo-Note) and [fzb.me](http://www.fzb.me/apollo/). Apollo has [a forum](https://talk.apolloauto.io/) which is not very active, it is recommended to ask questions as GitHub issues. A couple of instant-messaging options are available, mainly the apollodev wechat channel where discussions happen in chinese or the \#apollo IRC channel on freenode which is unfortunately not very active. One can also find classes documentation on doxygen.

The [overall structure of Apollo](https://github.com/ApolloAuto/apollo/blob/master/docs/specs/Apollo_3.0_Software_Architecture.md) is similar to any autonomous driving framework. It is quite nicely presented in a [free Udacity class about Apollo](https://www.udacity.com/course/self-driving-car-fundamentals-featuring-apollo--ud0419). We will remind the main ideas here:

At a low level, drivers provide access to the hardware. We can distinguish CAN drivers which control the car, LIDAR drivers which reconstruct laser measurements as a 3D point cloud, camera drivers which publish images based on the camera input, radar drivers which publish measurements similar to pointclouds and GNSS driver which publishes latitude and longitude data but also keep track of the precision of those measurements and keep the INS sensor connected to the NTRIP server.

On top of those low-level bricks, we can find a few crucial components. The perception module fuses sensor data and applies computer vision models to detect objects. The prediction module tracks objects over time and tries to predict their trajectories. The localization module also fuses information from different sensors to keep track of the "ego-vehicle". A high-definition map module is used for the car to be aware of its surroundings. A planning module uses the HD map, localization and prediction data to compute a trajectory for the car. A control module generates low-level commands \(throttle/brake/steering\) to maintain the car on the trajectory outputted by the planning module. A guardian module is used to monitor the status of the system and can trigger an emergency brake. A monitor module is responsible for restarting modules on a problem. Finally, a HMI module exposes information about the system status on a webpage called DreamView.

### Apollo Docker container

Everything related to Apollo runs in an isolated environment. Docker is a system that enables the creation of lightweight sandboxes called "containers". Containers are essentially virtual machines running a controlled system under a host operating system. Before doing anything with Apollo, the Docker container has to be started: \(this should be done automatically at boot\)

```text
apollo/docker/scripts/dev_start.sh
```

One can stop the Docker container using [a custom script](https://github.com/FlorentRevest/apollo/commit/72fbd19f98dff056e59679cea5205d1aed68e09f):

```text
    apollo/docker/scripts/dev_stop.sh
```

While the container is runnning, we can get shell accesses to the container using the following as many times as necessary:

```text
    apollo/docker/scripts/dev_into.sh
```

Apollo runs inside a Ubuntu 14.04 container which pre-installs all Apollo dependencies \(such as ROS packages or various Python modules using anaconda\). In this container, the apollo source code is setup to be mounted on /apollo/. Some data in Apollo are downloaded as Docker volumes, mounted on the source code and only available while inside the Docker environment. This is the case of HD Maps \(see _/apollo/modules/map/data_ inside and outside the docker environment\).

_Warning:_ Any change made outside of the _/apollo/_ directory is lost everytime the Docker container is restarted!

### Apollo build system

Apollo needs to be compiled from inside the Docker container. The build system used by Apollo is _bazel_ but it is encapsulated behind the _./apollo.sh_ script. Many build targets are included but the most important ones are _build_, _build\_opt_, _build\_opt\_gpu_, _build\_gpu_ and _build\_pandora_. The _\_opt_ suffix enables various compiler optimization and is recommended to be used. The _\_gpu_ suffix enables support for NVIDIA GPU, which is necessary for the prediction module to work \(DNN models running on CUDA\). The _build\_pandora_ target is [a custom target](https://github.com/FlorentRevest/apollo/commit/c59d4f6db450b69e8b7f3088c71fd7d7a48c9f0e) that you can use to compile the Pandora driver, usually not part of the rest of the build. This command [should be run everytime the docker container is started](https://github.com/FlorentRevest/apollo/commit/a6e6e1b925ddca1efc2759713c8008da78267f24) to avoid having the Pandora driver being wiped.

### Apollo message-passing infrastructure

As underlined in the introduction about Apollo, autonomous driving frameworks are made of a wide range of modules communicating with each others. Orchestrating those modules is a common task in robotics and is typically tackled by a framework called ROS \(the Robot Operating System\). Apollo has been using a slightly modified version of ROS up to Apollo 3.0 \(which we use\). The ROS inner-working has been changed to serialize data using Protobuf and to use shared-memory for large data such as point clouds but the API is still the same as the commonly used ROS.

A handful of concepts are necessary to understand ROS:

1. In ROS, every module/component is called a **ROS node**. For instance, the perception module is a node, the pandora driver is a node.
2. Node can exchange typed data over named communication channels called **ROS topics**. For instance, the perception node and Pandora driver can exchange 3D pointclouds on the _/apollo/sensor/velodyne64/compensator/PointCloud2_ topic which exchanged data of type _sensor\_msgs/PointCloud2_.
3. Data can be sent on a topic by a **ROS publisher** and received from a topic by a **ROS subscriber**.

Whole books are written about ROS but the above list summarizes its main idea very briefly. ROS is only the _plumbing_ behind Apollo runtime. By exposing communication channels and isolating modules as different processes, this makes the architecture of a robotic system very modular and flexible.

The best example of the flexibility of ROS is the _rosbag_ util. It can record data published on various topics in a file format called “.bag”. This can be done while some nodes are running \(for instance while the car is running autonomously\) using:

```text
    rosbag record -a test.bag
```

Then, those data can be transferred to another computer running ROS and replayed using

```text
    rosbag play test.bag
```

For instance, one can inspect the data published on each topic using graphic tools such as RViZ \(handy for pointcloud or image visualization\) or _rostopic_ \(handy for other type of data such as GNSS data, control commands etc...\)

One more example of the usefulness of ROS if the _/tf_ and _/tf\_static_ topics. This topic contains information about different frames used across the car. It describes a tree of transformation that connect different reference frames \(a bit like a 3D scene graph\) and can be used to infer new transformations between different points. Typically, this is used to describe extrinsic calibration across the car.

Other commands worth mentioning are _rosrun_ and _roslaunch_ used to start ROS nodes. _rostopic_ can be used to list and echo topics.

When using ROS, be careful to always run commands inside the Docker environment. Some commands can be run across the ROS container but can result in weird behaviors due to the ABI changes described above.

Starting from the version 3.5, Baidu developped their own alternative to ROS called CyberRT which is supposed to provide the same mechanisms but ensuring real-time. Since we can not use Apollo 3.5, we have not experimented with CyberRT yet.

### Apollo configuration engine

Each module in Apollo can typically be configured for a few things. For instance, one can change the topics the node will subscribe to. The configuration can be found in the folders _/apollo/modules/XXX/conf/_ where XXX is the name of the module you are interested in. Typically, a _XXX.conf_ file contains the arguments to be given by command line to the node. A _XXX\_adapter.conf_ file describes what topics the node will subscribe and publish to. Finally, a _XXX.pb.txt_ contains Protobuf data used for configuration of the module. The definition of each Protobuf data structure can be found in _/apollo/modules/XXX/proto/_.

### Apollo modules

Now that you know about the overall structure of Apollo, this section contains a description of each module and their use in the context of our own vehicle.

#### Common

The _common_ module contains various utility functions and structures used by the rest of the Apollo source code. For instance, a unified logging system is provided and can be used to debug your changes to Apollo with functions such as _AINFO \&lt;\&lt; ”logs”_. Other utils include common maths functions, filters, time functions, easy-to-access databases, general protobuf structures etc...

A part of _common_ worth noting is the _adapter_ submodule which abstracts ROS or CyberRT publish/subscribe code. All topics are hidden behind unified macros and all modules can automatically become publishers or subscribers to desired topics declared in their _conf/XXX\_adapter.conf_ file with a simple function call.

The _common_ module also contains a configuration file that describes parameters to be given to every Apollo module: _common/data/global\_flagfile.txt_. For instance, we use this file to [default DreamView to the navigation mode](https://github.com/FlorentRevest/apollo/commit/ee86e6815b7197d34f11ecaa80658bccf33a1856).

#### Monitor

The _monitor_ checks that nodes are running, that they publish their topics at a reasonable frequency and also performs some basic hardware health test, in our cases it checks the INS status and the CAN bus. This monitoring is then summarized in a low-frequency "system status" topic that is displayed in DreamView.

#### Guardian

The _guardian_ module uses the output of _monitor_’s system status to trigger a safety stop if a problem is noticed. First guardian generates a soft brake \(30% of brake\) and then moves on to an emergency brake \(50% of brake\).

#### Pandora driver

The Pandora driver is located under _/apollo/modules/pandora_ up to 3.0. In 3.5 it has been moved to a the [apollo-contrib](https://github.com/ApolloAuto/apollo-contrib) repository. The driver is split in two nodes.

The first node, _pandora\_driver_ is developed by Hesai \(the company behind the Pandora\) and is based on the code found in the [HesaiTechnology/Pandora\_ros](https://github.com/HesaiTechnology/pandora_ros/) repository. We [upgraded the code in Apollo to the latest version of Pandora\_ros](https://github.com/FlorentRevest/apollo/commit/e90737bb752afe0b036144457e59d25ed6bb46f6) otherwise cameras would not work. This node establishes a TCP/IP communication with the Pandora, retrieves the camera and Pandar40P data and publishes them on various topics. We modified this node to [remove handling of cameras not used by Apollo 3.0](https://github.com/FlorentRevest/apollo/commit/0c03ac6c14d9ea1b15c83ad7f77e4085e9b20e97) \(the back and rear cameras\). Please note that those cameras would be necessary in 3.5.

A few changes were also made to the format in which data are published: for instance, images were [published as rgb while they were actually bgr](https://github.com/FlorentRevest/apollo/commit/a302b6b0f42fdd0bd57418d1cf819341eaa06128) which resulted in weird colors in DreamView. Camera images [were published on topics that would not be used by the Apollo perception module](https://github.com/FlorentRevest/apollo/commit/f02f7cfe2a10dfe98363f76a8382a273e6724e4f). Apollo also makes the assumption that the LIDAR point clouds’ X axis points to the front of the vehicle while [the driver used to publish point clouds with the X axis pointing to the left](https://github.com/FlorentRevest/apollo/commit/d5e9f5bd3de2a9c63c835db885953609086231d0).

The second node, _pandora\_pointcloud_ takes the raw pointcloud outputted by _pandora\_driver_ but compensates the car movement. Since the vehicle is moving while the LIDAR rotates, one rotation of the LIDAR produces distances that are distorted. We [updated the code _pandora\_pointcloud_ based on the code of _velodyne\_pointcloud_](https://github.com/FlorentRevest/apollo/commit/57e075d4672c0f03eb694d23ad529b92ff5cc4de). Correcting this movement requires information from an IMU but also a very precise synchronization of each points with IMU data. This is the reason why the PwrPak7 is connected to the Pandora with a custom PPS cable, this is used to ensure that IMU and point clouds data have timestamps that correspond precisely to each others. Please note that the timestamp sent over TCP/IP by the Pandora is UTC while the timestamp sent over ROS topics by the GNSS driver is expressed in the local timezone. The _pandora\_driver_ has a timezone option to convert UTC timestamps to local timestamps, [we configured it for Norway](https://github.com/FlorentRevest/apollo/commit/ec9c06e80eea35d1ccc2a494a40f6aee353e0789).

#### RADAR driver

The driver for our RADARs is located under _modules/drivers/radar/conti\_radar_. It simply retrieves data that the RADAR publishes on the CAN Bus and publishes them in the appropriate ROS topics. More details on the CAN Bus in the next section.

#### CANBus

A critical component of any car built since the 90’s is the CAN Bus \(Controller Area Network\). The CAN bus is used to interface various ECUs \(Engine Control Units\) in a vehicle. Each electronic component can publish _CAN frames_ which are packets of data identified by an ID and that contain some data, they are broadcasted to all other ECUs. By reading information published by the car on its CAN bus, one can get insights on the status of each component. By publishing information on the CAN bus, one can control various functionalities of the car \(such as horn/traffic lights etc...\).

The KIA Niro, like most vehicles in the market, has a CAN bus on which components integrated by KIA communicate. We will refer to this bus as the "Vehicle Bus". In addition to this bus, the drive-by-wire system that we installed on our car, the DriveKit, has a secondary, isolated, bus that we will call the "DriveKit" bus.

The IPC does not have CAN controllers \(a chip that allows the IPC to read and publish data on the bus\) included to its facade, it is then necessary to have an external CAN controller. The DriveKit is distributed with a USB CAN controller manufactured by Kvaser. This USB CAN controller has a Linux driver that abstracts the controller with the standardized SocketCAN API contributed by Volkswagen to the Linux kernel. Provided that the driver is loaded \(modprobe kvaser\_usb\), one can then use SocketCAN to interact with both the Vehicle and DriveKit buses via the can0 and can1 network interfaces. If your IPC is connected to the KVaser USB controller, you should see a can0 and a can1 interfaces when running _ifconfig_. Apollo recommends using a PCI CAN controller \(the ESD CAN card\) because USB can not satisfy real-time constraints, however, this shouldn’t be a problem in our case since the DriveKit itself publishes frames on the vehicle bus in real time.

The Apollo CANBus module is split in three parts: the driver, that publishes or retrieve CAN frames on different types of components. The API which abstracts the drivers and can be used by various modules needing to access CAN buses \(for instance the RADAR driver\) and the vehicle implementation which translates steering/throttle and brake commands into CAN frames and publishes a "chassis" status, containing information about the vehicle such as its speed or current steering angle.

Apollo 3.0 already has a SocketCAN driver which means that Apollo can read and publish data on Linux canX network interfaces like the ones exposed by our USB controller. However, the way Apollo is architectured, it makes the assumption that only one CAN bus is used, and only publish on either can0 or can1. We then had to [modify the CAN bus module to handle two CAN buses](https://github.com/FlorentRevest/apollo/commit/aeb8256bb3b2093b77706c082b24b189ee71cfa8#diff-c47a88fa12006925292435d82c4e8336). With this patch, frames are always published on the primary CAN bus and the secondary CAN bus is only used as a receiver. The configuration of what can bus is primary of secondary is done in [_canbus\_conf.pb.txt_](https://github.com/FlorentRevest/apollo/commit/43794def716b4a4ef476a355e7f315da47dbbf6b#diff-c47a88fa12006925292435d82c4e8336)

Another limitation of Apollo is the assumption in its API that CAN frames shall be run periodically. However, in the case of our DriveKit, some frames should absolutely be ran once. For instance, the throttle enable/disable commands should not be run periodically or the DriveKit would keep engaging/disengaging. We then had to [extend the CAN Bus API to allow sending messages once](https://github.com/FlorentRevest/apollo/commit/1deb42d500c99974ddf21cb4f9e194a74535d3af#diff-0eb9b3af2e4a00837a1b1a854c9ea18c).

On top of this API, we had to implement a vehicle module that generates the CAN frames expected by the DriveKit based on control commands. This module is partly code-generated thanks to _.dbc_ files distributed by PolySync and transformed into C++ code by the _gen\_vehicle\_protocol_ tool \(more details on this later\), however a lot of manual fixing had to be done. For instance, the generated code would not handle both CAN buses, so two results of _gen\_vehicle\_protocol_ had to be run. Also, every unnecessary CAN frames was removed, the code of some modules was re-worked for clarity or to just work. An important implementation detail is the usage of steering angle control, which is a command provided by the DriveKit which takes a steering angle target and maintains a PID controller that generates corresponding torque commands to maintain the steering wheel at a certain angle.

PolySync distributes on their GitHub webpage a roscco-apollo module, which is a ROS node aiming to replace the whole apollo can bus module, by communicating with roscco, a ROS module interfacing with the OSCC API, a library that communicates with the OSCC firmwares. However, this whole stack was judged unnecessarily complex and could introduce undesired latency. Also, those modules were written with OSCC, the open-source version of the DriveKit, in mind, which does not have the steering angle control commands we just mentioned. This module would therefore have to keep its own PID controller to keep track of the steering torque and various details led us to believe that [this code was actually never tested](https://github.com/PolySync/roscco/issues/14). We then rewrote a module "correctly" instead of relying on a pre-written module.

#### GNSS driver

The GNSS driver is located under _/apollo/modules/gnss_. Despite its generic name, it is quite specific to Novatel devices. It communicates over virtual serial lines over USB \(_/dev/ttyUSB\*_ devices\) with the PwrPak7.

One might think that the communication between the PwrPak7 and the IPC would be one-way only but that is wrong. In order to achieve centimeter-level accuracy, the PwrPak7 uses an RTK \(real-time kinematic\) system to enhance the precision of its positioning. This system communicates with a base station using NTRIP \(Networked Transport of RTCM via Internet Protocol where RTCM means Radio Technical Commission for Maritime Services\). The acronymes make it look more complex than it actually is. Essentially, an antenna is placed somewhere near the vehicle and knows its position with very high precision, it computes in real time the difference between the carrier phase of the signal emitted by GNSS satellites and its actual position \(caused by geomagnetic activity for instance\), then it publishes this difference over the Internet to allowed NTRIP subscribers. Apollo has an NTRIP client that sends the correction data as RTCM signals to the PwrPak7 which uses this to improve the carrier phase it receives from satellite and can deduce a much better positionning. The NTRIP caster we use is located on a hill near Trondheim and is maintained as part of the EUREF network of the Royal Observatory of Belgium.

The PwrPak7 computes its INS positioning by fusing data from its IMU and GNSS. This process takes time to converge and requires the vehicle to move. The status of INS positioning is published on the _/apollo/sensor/gnss/ins\_status_ topic and can also be viewed from the DreamView HMI in the "Modules" tab as the GNSS status indicator.

In Apollo, localization data are not expressed in terms of latitude/longitude, they are expressed as x/y coordinates in UTM Zones. UTM Zones conveniently divide Earth in 6 degrees-wide bands... Except in Norway. The south of Norway \(including Trondheim\) and Svalbard are exceptions to the way UTM zones divide the world. However, they seem to be handled correctly by Apollo. The GNSS driver is configured to publish data in our UTM Zone, the number 32.

The GNSS driver in Apollo 3.0 had an incompatibility with the IMU inside our PwrPak7, the Epson G320N, but we fixed it in our code and sent a patch upstream.

#### Other drivers

The _drivers_ module directory contains lots of drivers that we currently do not use. For instance, [Velodyne LIDAR](https://velodynelidar.com/) drivers, [Leishen LIDAR](http://en.leishen-lidar.com/index.html) drivers, [RoboSense LIDAR](http://www.robosense.ai/) drivers or USB cameras drivers. Most of those drivers are just based on the ROS driver provided by the constructor so it should be fairly easy to add new drivers if we someday decide to use new sensors such as [Ouster LIDARs](https://www.ouster.io/).

#### Maps

Maps such as the ones provided by Google Maps or OpenStreetMap are not enough for autonomous driving. Instead, autonomous systems rely on High-Definition maps \(HD maps\). Two modules of Apollo require two different types of HD maps.

The _localization_ module requires localization maps. They are described in the next section about the _localization_ module and stored in a custom format defined by Baidu.

The _planning_ module uses a different type of map which has very precise metadata describing each possible lanes in which the car can drive, where traffic lights or crosswalks are expected to be situated etc... Those maps are stored in an XML format derived from the OpenDrive standard defined by VIRES. The specifications of this format are not publicly distributed for unclear reasons but we got them from Baidu by [emailing them](https://github.com/ApolloAuto/apollo/issues/4048). Those specifications are available for you to download here. The code parsing Apollo OpenDrive maps, in _modules/map/hdmap/adapter/_ should also be quite helpful for parts that are left unclear by the specs \(eg: lat/lon -&gt; xy conversion or geometry representations\).

HD Maps can be visualized using tools described in the _tools_ section.

Apollo has a “_navigation_” mode in which having a prior map is not necessary. It creates a [relative map](https://www.youtube.com/watch?v=1WSGlFK99q8) in real-time by trying to detect lanes using its perception module. However, Baidu told us that relative-maps only work on freeways which are obviously not the safest places to test on.

Creating an HD Map is a tough problem. At this point, we identified several ways to create HD Maps:

1. _Baidu method:_ Baidu recommends putting together an HD Map acquisition vehicle, with two LIDARs: one 16-beams and one 64-beams. This vehicle needs very precise extrinsics. Then bag recordings can be sent to a Baidu cloud service that will first try to automatically generate an HD Map and then a mapping team from Baidu will manually correct the map. Baidu told us that this service costs 2000$ per km and per lane. 10 km of three-lane highways then costs one million NOK. It is also worth noting that Baidu has limited interest in mapping Norway and could also simply refuse our mapping request.
2. _Precivision method:_ There are tons of HD maps company out there but one of them stands out by showing off a demo of their cartography system working with Apollo on their website. We have not been in touch with [Precivision](https://www.precivision.tech/) yet but they might be worth contacting.
3. _Mobileye method:_ Apollo has a tool called _create\_map_ which uses the lane marking recognition of Mobileye sensors to create HD Maps. At this point, it is unclear what would be missing from those generated maps and what we could do about it.
4. _Planning map generation from trajectory:_ Given a trajectory, for instance acquired through RTK or using APIs of online services such as OpenStreetMap or Google Maps, one can automatically generate a fake dummy planning map with imaginary lane markings. This is what the _map\_gen_ tools do. For an example of how to use map\_gen, you can refer to _scripts/create\_map\_from\_xy.sh_ or to the following few lines:

   ```text
       python modules/tools/map_gen/extract_path.py moholt.xy moholt.bag
       scripts/create_map_from_xy.sh --xy /apollo/moholt.xy --map_name moholt
       scripts/msf_simple_map_creator.sh /apollo/data/bag/ /apollo/modules/calibration/data/kia_niro/velodyne_params/velodyne64_novatel_extrinsics_example.yaml 32 /apollo/modules/map/data/moholt
       mv /apollo/data/bag/lossy_map /apollo/modules/map/data/moholt/local_map
   ```

5. _Creating our own HD map generation tool:_ We could potentially use sensors acquisition \(LIDAR/Cameras\) and do offline 3D reconstruction and object reconstruction to automatically output a map of the environment usable by Apollo but this could require a lot of work.
6. _Rewriting the Apollo modules:_ A big concern with the way Apollo maps work is that they rely strongly on visible lane markings \(for instance used in the LIDAR intensity map in MSF\) which are often missing or hidden under snow in Norway. One approach could be to rewrite new localization and planning modules using different sources of data, that would work better in snowy conditions. There are already some research done on the topic of [localization in snowy environments](https://ieeexplore.ieee.org/document/7844000).

#### Localization

The _localization_ module aims at keeping a precise position of the vehicle over time. Three localization methods are available.

The _localization_ module can be configured in _RTK_ \(Real Time Kinematic\) mode, where it solely relies on the output of the GNSS driver to determine its position. Although the accuracy of an RTK system should stay within one-centimeter, this is prone to sudden changes and can stop working in areas that are covered \(for instance, in a tunnel\).

The _localization_ module can also be configured in _MSF_ mode \(Multi-Sensory Fusion\). This method is extensively [detailed in a paper published by Baidu](https://arxiv.org/abs/1711.05805). The basic idea is to have a pre-recorded LIDAR intensity map of the area where the car should drive. Then, MSF uses an IMU and RTK to get an estimate of the vehicle’s position, and aligns the currently observed pointcloud with the previously recorded pointcloud to get a precise positioning. [A YouTube video](https://www.youtube.com/watch?v=8wRs_TaAfUk) gives a fairly good understanding of the benefits of MSF over RTK-only localization \(especially after 30 seconds\). You can reproduce the result of this video by [running MSF locally](https://github.com/FlorentRevest/apollo/blob/v3.0/docs/howto/how_to_run_MSF_localization_module_on_your_local_computer.md).

Finally, Baidu provides a module named _elo_ which can act as a replacement for _localization_. _Elo_ needs to be downloaded externally and requires support from Baidu. This module also uses RTK and IMU to get an estimate of the vehicle’s position but it then uses the ENet DNN to recognize lane markings on the ground and aligns those features with a localization HD Map distributed by Baidu.

At this point, since we do not have a good MSF map, [we use RTK localization](https://github.com/FlorentRevest/apollo/commit/34670de41f1e5ced2d3ee800b8591ff3d470462c).

#### Perception

Probably the most important module for our laboratory is the _perception_ module. This module is very flexible and is configured at a high level with a Directed Acyclic Graph \(DAG\). The graph describes the data flow from sensors to the perception output. Each node of the graph produces, processes or outputs data. Different example DAGs are provided in its _conf_ directory. The two main ones are:

1. The standard one, _dag\_streaming.config_ which fuses information from the LIDAR, radar and traffic lights camera \(only when the HD Map indicates a traffic light being close-by\).
2. The "low-cost" perception, _dag\_streaming\_lowcost.config_ which fuses information from a camera and radar.

Other DAGs can be used to visualize the output of perception nodes, suffix _\_vis_. Or to process data off-line, from the lab, suffix _\_offline_. For instance, [one can visualize the LIDAR perception](https://github.com/ApolloAuto/apollo/blob/master/docs/howto/how_to_run_offline_perception_visualizer.md) or one we can also [visualize the LIDAR+RADAR fusion](https://github.com/FlorentRevest/apollo/blob/v3.0/docs/howto/how_to_run_offline_sequential_obstacle_perception_visualizer.md).

In navigation mode \(more information about this in the DreamView section\), Apollo is configured to use the low-cost perception DAG. Since we do not have an HDMap, we only use the navigation mode and currently do not use the LIDAR for perception.

DAG nodes processing camera information use CNNs developped with Caffee, they require CUDA to run.

[Details information on the architecture of the perception module](https://github.com/ApolloAuto/apollo/blob/master/docs/specs/perception_apollo_3.0.md) can be found in the Apollo documentation. [Another page details the inner working of the 3D object detection](https://github.com/ApolloAuto/apollo/blob/master/docs/specs/3d_obstacle_perception.md). Finally, [a page describes the traffic lights detector](https://github.com/ApolloAuto/apollo/blob/master/docs/specs/traffic_light.md).

No change were made to this module.

#### Prediction

Another important module for NAPLab members is the _prediction_ module uses the output of the perception module and lanes information \(if available\) to estimate the trajectory of objects in the scene \(people/cars/bikes/etc...\). Different methods are available based on a RNN, MLP and cost-evaluator models.

[Instructions on how to re-train the MLP prediction model](https://github.com/FlorentRevest/apollo/blob/v3.0/docs/howto/how_to_train_prediction_mlp_model.md) are provided in the Apollo documentation.

No change were made to this module.

#### Routing

The _routing_ module generates very high-level routing information. For instance, this module requests the car to go from a point A to a point B.

#### Planning

The _planning_ module uses the output of the _routing_ module, information available in a _HD map_ and obstacles position and trajectory outputted by the _prediction_ module to create a precise trajectory of where the vehicle should drive. For instance, this module can plan in which lane to stay, or when to change lane, if it makes sense to do so.

Please note that the above mode is called EM-planner and is [documented in a paper by Baidu](https://arxiv.org/abs/1807.08048). The module can also be configured as an RTK-planner where it only replays a trajectory.

#### Control

The _control_ module uses the output of the _planning_ and _localization_ modules, but also the "chassis" status published by the _canbus_ module and tries to generate low-level control commands \(throttle/brake/steering angle\) to maintain the car on the planned track while avoiding obstacles in the lane.

The control module can work in two configurations:

1. A separate lateral LQR controller \(for the steering\) and longitudinal PID controller \(for the throttle and brake\)
2. A MPC controller that handles both

We currently use the same configuration as the one used for the Lincoln MKZ \(the Apollo reference vehicle\), which relies on separate lon and lat controllers. [Details on how to tweak the control configuration](https://github.com/FlorentRevest/apollo/blob/v3.0/docs/howto/how_to_tune_control_parameters.md) are provided in the Apollo documentation.

Please note that the output of the control module is not directly forwarded to the CAN Bus module. The guardian module is a layer placed in-between those two modules that either forward control commands or overwrites them with a safety stop.

#### DreamView

Most modules in Apollo do “behind-the-hood” work, by taking data on ROS topics, processing them and then outputting new data on other ROS topics without anything being actually visible to anyone. To keep an overview and control over the system, one module is only dedicated to presenting information to the user in a user interface, _DreamView_.

DreamView is a web interface split across a C++ ROS Node backend an a React/Webpack/Threejs frontend written with standard web technologies. When started, DreamView is accessible from a web browser at the port 8888, we typically access it from the iPad in the front of our car.

The UI of DreamView is very well [described in the Apollo documentation](https://github.com/ApolloAuto/apollo/blob/master/docs/specs/dreamview_usage_table.md).

Among the changes we applied to DreamView, we [replaced the Baidu Maps mini-map](https://github.com/FlorentRevest/apollo/commit/2e57da45172cc661913e905b041496f8820ddd0c#diff-cbaf11079bc17fb77ee270076797ce4f) shown in Navigation mode with a Google Maps mini-map which is more helpful in Norway. We also [fixed the compatibility of the dashcam functionality with the Pandora](https://github.com/FlorentRevest/apollo/commit/129ab704a707ecfc1cae6054c619e881430e4044#diff-cbaf11079bc17fb77ee270076797ce4f) and [modified the position of the Dashcam](https://github.com/FlorentRevest/apollo/commit/20a92ccb7c83f23ccd040b5362745cb2a9cad9c0#diff-cbaf11079bc17fb77ee270076797ce4f) for it not to overlap with the Google Maps mini-map shown in Navigation mode.

An important configuration file for DreamView is _conf/hmi.conf_ it describes all the modes that can be selected and what modules can be started from each of those modes. We modified this file [to expose the Pandora Driver from the Navigation mode](https://github.com/FlorentRevest/apollo/commit/8aebd009247c6184aa0172a5001c6298c4d2352c#diff-cbaf11079bc17fb77ee270076797ce4f) and also to [hide some components that we do not use like external cameras or third party perception](https://github.com/FlorentRevest/apollo/commit/4ce0fb6856451c169f34a7cb35828180ada22233#diff-cbaf11079bc17fb77ee270076797ce4f). Please note that most of the actions triggered by the DreamView HMI are actually bash scripts located in _/apollo/scripts/_ which in turn load their corresponding ROS nodes with the necessary options.

#### Calibration

Apollo depends on a few vehicle-specific configuration files. Typically, calibration data such as intrinsics, extrinsics, control calibration and vehicle definition \(more details on the nature and acquisition of those different types of data in the next section about Calibration\).

Each vehicle calibration files are stored inside a subdirectory of _modules/calibration/data/_. The names of the expected files are listed in _modules/dreamview/conf/vehicle\_data.pb.txt_ along with the destination at which they get copied when someone selects the corresponding vehicle in DreamView’s top bar.

#### Tools

The _modules/tools_ directory contains a lot of subdirectories covering many use cases you might have.

[The _calibration_ tool](https://github.com/FlorentRevest/apollo/blob/v3.0/docs/howto/how_to_update_vehicle_calibration.md) is used to acquire throttle/brake calibrations. It generates different throttle and brake commands \(in place of the control module\) and records the output of the localization module. Those data can then be post-processed to generate calibration curves usable by the actual _control_ module. The _data\_collector.py_ script takes three numbers: a throttle command, a maximum speed and a brake command to output when the maximum speed is reached, until the car stops. However, this script does not seem to work so [we incorporated a script from Apollo 1.0](https://github.com/FlorentRevest/apollo/commit/8cf23151eda9d36e4602280ec4aae54c75d2a600) named _data\_collector\_old.py_ which takes as argument a .txt file \(multiple examples can be found in _calibration\_data\_sample_\) and can do more complex scripting.

The _configurator_ can be used to edit protobuf configuration of various modules. This is equivalent to editing the files by hand and hence not very useful.

The _controlinfo_ script can be used to debug the behavior of the control module provided a planning command.

The _create\_map_ directory contains two scripts that can generate localization HDMaps for people who have a Mobileye sensor. They can use _lane\_recorder.py_ to record a sequence and then _create\_map.sh_ to generate the corresponding OpenDrive file.

The _diagnostic_ tool can be used to verify the frequency at which various topics are published. It is equivalent to the bottom center view of DreamView’s main screen.

The _extrinsics\_broadcaster_ tool reads TF transform stored in a Protobuf format and publishes them on the _/tf_ ROS topic.

The _gen\_vehicle\_protocol_ tool reads .dbc files that contains description frames acceptable on a CAN bus and tries to generate a CANBus vehicle module. You can find in our fork of Apollo two .dbc files which correspond to both the Vehicle and DriveKit CAN buses. The tool was ran on each file and then the output was modified to get the module properly working.

The _lattice\_autotuning_ tool has never been tried but it seems to be used for calibrating the _control_ module.

The _localization_ tool can be used to [compare the performance of RTK and MSF localization](https://github.com/FlorentRevest/apollo/blob/v3.0/docs/howto/how_to_run_MSF_localization_module_on_your_local_computer.md#7-verify-the-localization-result-optional).

The _manual\_traffic\_light_ tool can be used to monitor the status of each traffic lights recorded in the currently used HD Map, or those which are close to the current localization. It can be used to debug if the perception module properly updates the color of a traffic light.

The _map\_gen_ directory contains a variety of tools that can generate fake high definition HD maps. For instance, one can extract the trajectory of a .bag file using _extract\_path.py_, then use _map\_gen\_single\_lane.py_ to create an OpenDrive map that contains procedurally generated lane markings. This map can not be used for precise localization but can be used in very limited scenarios for instance when testing a RoI filter \(region of interest\). A few scripts can also manually generate traffic lights. Those scripts can also be useful to look into for our own research into Apollo HD Maps.

The _mapshow_ tool can be used to visualize an OpenDrive HD Map.

The _mobileye\_viewer_ tool does what its name suggests and has never been used because we do not have Mobileye sensors.

The _mock\_routing_ script asks the planning module to generate a trajectory between two hardcoded waypoints. Their coordinates do not correspond to anything of interest in our UTM Zone but we could tweak this script to our own needs.

It is a bit unclear what the _navigation_ module is about, with scripts that seem to be very application-specific, with references to mobileye sensors etc. We do not believe that directory would be of any use to NAPLab.

The _navigator_ directory contains scripts to record a trajectory, smooth it out and send it as a navigation request to the other modules.

The _ota_ directory contains scripts related to Over-The-Air updates that is only available to Baidu partners and not of interest to us.

The _pandora\_camera\_scaler_ has been written by us to accomodate the needs of Baidu extrinsic calibration tools. If one wants to run the Baidu camera-camera, camera-lidar or camera-radar proprietary tools, images have to be in a certain format. This conversation is too heavy to be done in-line so we created this script that can reformat images stored in a .bag recording. The new .bag can then be played and is accepted by the Baidu tools.

The _pandora\_pointcloud\_renamer_ is also a home-made script that renames a _/apollo/sensor/velodyne64/compensator/PointCloud_ topic into _/apollo/sensor/velodyne64/VelodyneScanUnified_ in a bag. This topic name is expected by the GNSS-LIDAR extrinsic calibration service in the Baidu Cloud. However, we can not use this service from Europe so this script should be of limited use.

The _perception_ tool generates fake perception data, to simulate the perception module during development time.

_Planning\_traj\_plot_ plots planning trajectories and localization data recorded in a prototbuf file.

textitPlot\_control is used to plot steering/throttle/brake commands.

The _plot\_trace_ tool is used to plot and compare a pre-recorded trajectory \(while driving manually\) with an autonomous trajectory \(drawn while Apollo drives\). \(See _record\_play_ for more information on RTK replay\)

The tools in _prediction/mlp\_train_ are used to train and manipulate .h5 DNN models for the perception module.

The _realtime\_plot_ tool shows information about planning and control

The _record\_play_ tools are used for RTK replay. One script can record a trajectory from the localization module and the other can generate a planning message based on the recorded trajectory. This is useful to debug the control module.

The _relative\_map\_viewer_ plots map as they are discovered through sensors in navigation mode.

The _replay_ tool is quite similar to _record\_play_ except that it records planning data.

The _rosbag_ directory contains a bunch of tools related to _.bag_ files processing. For instance, scripts are provided to get statistics about the "bumpiness" or the mileage, of extrinsics in a _.bag_. Other scripts can dump various info from a _.bag_ such ass planning/gps/images/trajectory/planning/... data.

The _routing_ directory contains scripts related to the visualization of maps and routing data.

The [_supervisord_](http://supervisord.org/) directory contains configuration for the supervisor which in in charge of starting or restarting modules based on some policies.

The _voice\_detection_ contains a simple speech recognition module that can be triggered from DreamView and responds to the commands "Apollo, setup", "Apollo, auto-mode" and "Apollo, disengage".

_Note:_ All tools plotting things use matplotlib which is not correctly installed by Baidu inside the Docker environment. Also, this requires an Xorg server to be installed on the IPC to visualize data. We voluntarily did not install an Xorg server to keep ressources for the autonomous driving system. You might then want to run those tools on a different machine than the IPC itself.

#### End-to-end

The source code of Apollo contains an "e2e" module folder which stands for end-to-end model. However, this folder only contains a README with a link to an archive to download. This archive is not maintained and has apparently never worked. It could maybe still be used as an inspiration for some of the work being done at NAPLab. We do not use this module.

#### Data

Apollo has a _data_ module which is apparently used by some Baidu partners to automatically collect and store data. However, this module clearly lacks documentation and does not seem useful nor usable to NAPLab. We do not use this module.

#### Third party perception

A module called _third\_party\_perception_ could be used as an alternative to the _perception_ module described earlier. This module interfaced with sensors providing their own perception information such as Mobileye sensors or Continental RADARs. However, the features of this module were merged into the _perception_ module in 3.0 and it is not recommended to use this module. We do not use this module.

### Important directories

Every module in Apollo outputs different level of logs \(info/warning/error\) into files located in the _data/log_ directory. When you notice that a module does not stay active or if you observe weird behaviors, you are strongly encouraged to check the files in that directory first.

DreamView has a "_Record bag_" module that can be used to avoid having to run _rosbag record_ manually. The bags recorded by that module are stored in the _data/bag_ directory.

When a module crashes, core dumps are outputted in the _data/core_ directory. This directory can quickly becomes really big. If you notice you will quickly run out of disk space, it is recommended to delete this directory.

## Simulator

While developing an autonomous driving system, it is safer to test driving scenarios without having to run anything on an actual car and in real traffic conditions. For that purpose, we use virtual environments and let the autonomous driving system run in the equivalent of a game. A 3D world is rendered whose images are sent to the different camera input of the autonomous systems, point clouds are synthesised and sent to the LIDAR input of the autonomous driving system etc... Various autonomous driving simulators are available on the market but the one used at the lab is CARLA, an open-source simulator based on Unreal Engine.

CARLA has a Python API which allows to both control the simulator and retrieve data from it. We wrote an example script that spawns a vehicle conceptually similar to our KIA Niro, with 5 cameras and a 40 beams LIDAR similar to the ones used by our Pandora. This script is [available on GitHub](https://github.com/FlorentRevest/carla/blob/master/spawn_niro.py). However, please note that the extrinsics are not the same as the ones used in our car.

The version of CARLA we worked with was 0.9.3. At the time of this release, a bunch of limitations were identified. First of all, the simulator [did not support radars](https://github.com/carla-simulator/carla/issues/1331). Furthermore, the GPS coordinates emitted by the simulator were not in a format understandable by Apollo. Indeed, CARLA comes with a "ROS bridge" that emits simulated sensor data onto ROS topics however the GPS signal was a NavSat message while Apollo expects special RTK messages with additional information. CARLA also [did not support an IMU](https://github.com/carla-simulator/carla/issues/1029). Finally, Apollo also relies on HD maps in a Apollo-specific OpenDrive format and CARLA publishes maps in a standard OpenDrive format.

Despite those limitations at the time of writing, it is expected that CARLA evolves quickly and should provide those features in a near-future. We therefore spent some time trying to integrate what could be integrated and found out about this setup that worked to send data from CARLA to Apollo and back: One needs to run CARLA outside of the Apollo Docker container, the CARLA ROS Bridge inside of the Apollo Docker container, with the host’s CARLA TCP port exposed to the container and one also needs to run [a Apollo-CARLA Ros bridge similar to the one developed by Nemodrive, a romanian lab](https://github.com/nemodrive/apollo/tree/master/ws_apollo_carla_bridge/src/apollo_carla_bridge).

In the light of all those limitations, we should also highlight that the [LG SVL Simulator](https://www.lgsvlsimulator.com/) seems to offer a considerably better integration to Apollo, with a specific Apollo bridge and features such as HD Maps generation and editing.

## Calibration

#### Sensor calibration

An important aspect of setting up a robotic system such as an autonomous car is to calibrate all sensors. One can distinguish two different type of sensor calibrations: intrinsic calibration and extrinsic calibration.

Intrinsic calibration measures properties that are specific to the sensor itself. For instance, a camera has geometric distortion that can be measured and corrected. Our vehicle has 5 cameras that need calibration before being usable by the perception module \(for instance to estimate distance of objects or to avoid distortion in angles\). Getting intrinsics for a monocular can be done using [a ROS Node recommended by Apollo](http://wiki.ros.org/camera_calibration/Tutorials/MonocularCalibration). This node uses checkerboard patterns. We printed a small checkerboard image stored in the lab’s cupboard but it is not large enough to provide good results. It was recommended by members of Ascend NTNU to use [Kalibr](https://github.com/ethz-asl/kalibr) instead, with aprilgrid patterns which work better in the borders. Ascend has one of those aprilgrid patterns and we should be able to borrow it from them if we need it a contact to that company is listed in the corresponding section.

Extrinsic calibration measures relative poses between sensors. For instance, to do cameras fusion, one needs a precise measure of both the distance and orientation between the two cameras. The PwrPak7 needs an extrinsic calibration between the GNSS antenna and the IMU. Apollo requires a few extrinsic measurements: camera-camera, camera-lidar, radar-camera, lidar-imu and imu-vehicle. Baidu provides a bunch of proprietary pre-compiled tools to automate the extrinsic calibration necessary for Apollo. Some of the tools work, for instance, the LIDAR-GNSS calibration uses [the “hand-eye method”](https://github.com/FlorentRevest/apollo/blob/v3.0/docs/specs/lidar_calibration.pdf) on a bag where the car drives in an 8-shape to optimize an accumulated pointcloud and deduce a precise pose. However, other tools have been found to be incompatible with the Pandora \(due to a wrong image resolution, wrong pixel format that we fixed off-line using a python script described in _modules/tools_ but also, it seems, wrong number of laser beams\). This was [reported to Baidu](https://github.com/ApolloAuto/apollo/issues/7840) without progress so far.

Since the Pandora does not seem to be compatible with the extrinsic calibration tools provided by Baidu and we do not have a good checkerboard or aprilgrid pattern, we decided to extract calibration data done by HESAI and stored inside the Pandora. Those calibration data can be [retrieved by tools such as _pandora\_projection\_ros_](https://github.com/HesaiTechnology/pandora_projection_ros/blob/master/src/pandora_projection.cc#L537).

Other extrinsic calibration, such as the GNSS-IMU distance were found using an optic method. We placed a reference distance on the ground, in front of the car, then took a picture of the car using a tele-lens. We then processed the picture using a software called [Fiji](https://fiji.sc/). In order to measure distances with components situated inside the trunk \(such as the IMU\), we built a lever arm of known dimensions that brings a clear point outside of the car. You can find this lever arm in the lab, right at the left of the cupboard.

_Note:_ Apollo uses different convention for different sensors:

1. The LIDAR should have its X axis forward, Y axis to the left and Z axis to the sky
2. The IMU should have its X axis to the right, Y axis forward and Z axis to the sky
3. Cameras should have their Z axis pointing forward, and X and Y axis pointing to the X and Y axis of the image

_Note:_ Handling and visualizing extrinsic calibration can be tough at times. We recommend publishing transformations on the ROS _/tf_ topic and visualizing them in RViZ. We also recommend tools such as the [online quaternion visualizer](https://quaternions.online/).

#### Algorithms calibration

The control module of Apollo requires throttle and brake calibration. This is done through the tool found in _module/tools/calibration/_ and already described above in the tools module section.

In order to detect collisions, Apollo also requires dimensions of the car. Those were obtained from [the Wikipedia page of the KIA Niro](https://en.wikipedia.org/wiki/Kia_Niro).

