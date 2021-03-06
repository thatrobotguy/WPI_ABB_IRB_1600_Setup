# ABB_IRB_1600_Setup
This is the repository that explains how to set up the ABB IRB 1600 robot arm at WPI's Washburn Labs and the ABB IRB 120 mini robot arm with ROS.

## Prerequisites
This guide assumes many things about the people reading it. This guide is not for those kinds of people without any experience with ROS. This guide refers to other guides so this tutorial is not complete in the traditional sense. I have to point you to the other links because they may get updated sooner then this document. Also, I do not maintain those packages, so I am not the one with all the answers. This document is simply how I got my machine working, but with much more clarity than scratch work and other [MQP git repositories](https://www.wpi.edu/academics/undergraduate/major-qualifying-project).

#### Things to have set up before starting on this guide
1. A working installation of Ubuntu 1604 on any computer
2. This computer must have an ethernet port. Theoretically, you could run this installation with a USB to Ethernet adapter, but we have not tried that.
3. ROS Kinetic
4. Python 2.7
5. Skills on the Linux terminal
6. Stable internet connection
7. git
8. Your favorite text editor

### Before you begin...
If anybody feels there are changes that need to be made to this repo, make a pull request or create a github bug thing or whatever it is called. Either I or somebody else may have seen the bug before. I have been making frequent updates so that this document and repository looks as clean as possible. I also should mention that I have tried to list the steps in order, but some steps _can_ be done out of order, and some steps may be in the wrong order, so you should read this document in its entirety before embarking on the installation of `ROS Industrial` and `MoveIt!`. I should also mention that, before blindly doing the steps in all of these tutorials, you need to read the steps I ask you to do before you do them. For example, multiple tutorials show different commands that have to do with modifying your `~/.bashrc` flie. I already provide most of the environment variables that are necessary to make ROS work, therefore you should only add environment variables when they have been __explicitly__ mentioned __outside__ of this tutorial  and __not__ explicitly mentioned __here__. Also, if I say to git clone something, assume that it needs to be `git clone xxx` in your catkin workspace.

## Updates

I will be updating this document periodically. I will pose those updates __TO THE BOTTOM__ of the document. Be sure to check those out before continuing on...

## Static IP configuring
In order to set a static ip address, you will first have to know some stuff.

For the networking portion of this tutorial, I am assuming that you are on a network that is managed by a business or school. If not, setting the static IP may require a different process. At home, I have to set the static IP on the router, not my desktop. Even then, I can only use simulation in this case. You will have to think about what you are doing before you go and do what these other links say to do. 

The first thing you will need to do is set up a static ip address on the ethernet nic on the machine that will directly drive the robot.

### Assuming you are using the setup in Washburn

In order to network with the robot, you will be connecting your device to an ethernet port on a network switch. You can directly interrface with the IRC5 ethernet port (the one that is _not_ the `LAN` port), but it is better to use a switch, especially if your cables are too short or if you need other sensors in the ROS local network. 

The ABB robot arm resides on a closed off local network. Even in manufacturing lines, the robot arms reside on their own local network. This means that the arm cannot _directly_ access the internet (as one would hope).
The device that attempts to connect to the arm should note that the arm expects to be assigned an IP address of `192.168.100.100`. This means that your machine _cannot_ be assigned this address. I gave my laptop a static IP of `192.168.100.123`. If you want to know what IP addresses are currently in use on the network, plug in to the switch and type this:
```
sudo arp-scan --interface=myethernetcardname --localnet
```
(I am also assuming that the network is Ethernet only as there usually is NOT a WiFi network available).

This command will list all of the IP addresses on the network that you have attached to your ethernet port on your laptop.
In order to find out what your ethernet IP is, just run `ifconfig`, or, if you want to use a more modern command, `ip addr`. You should see something like this on the output:
```
$USER@$USER-MyComputerModel:~$ ifconfig
enp7s0f1  Link encap:Ethernet  HWaddr jokes:on:you:bro
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:277219 errors:0 dropped:0 overruns:0 frame:0
          TX packets:277219 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:45637381 (45.6 MB)  TX bytes:45637381 (45.6 MB)

wlp0s20f3 Link encap:Ethernet  HWaddr not:for:you:sucka
          inet addr:192.168.0.23  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::a11f:ab53:e881:ad/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:20356 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12788 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:17042440 (17.0 MB)  TX bytes:2453717 (2.4 MB)

```
Given this output, I would type this to scan the local network for connected devices:
```
sudo arp-scan --interface=enp7s0f1 --localnet
```
If this errors out, you probably need to either install `arp-scan` or check your spelling.
I should also mention that the address you choose _must_ be in the range of `192.168.1xx.2` to `92.168.1xx.255`.
Now that you have figured out what IP addresses are currently is use, decide what IP address you would like to use on your laptop. 

You first need to click on the internet symbol in the upper right hand corner (furthest left icon):

![Image of Ubuntuwifi](https://github.com/thatrobotguy/WPI_ABB_IRB_1600_Setup/blob/master/upperrightcorner.png)

Now you need to select the ethernet option in the next menu like so:

![Image of network options](https://github.com/thatrobotguy/WPI_ABB_IRB_1600_Setup/blob/master/networkoptions.png)

Then you click `edit`. Now, go to the IPv4 settings tab and set the `Method` to `Manual`. now you need to enter the address and netmask and then click `add`. Set your machine IP like this:

![Image of ipv4 settings](https://github.com/thatrobotguy/WPI_ABB_IRB_1600_Setup/blob/master/ipv4_settings.png)

Now once these are set you will want to click `save` and then restart your machine to make sure all your changes are written properly.

## I am using a USB to Ethernet adapter of some kind. Does that work too?

Yes, you can use an adapter. The only issue I have experienced is that you have to set up the static `ip` every single time you boot your machine. This is just an issue when you don't have the best hardware for the job. If you are to recommend somebody to buy a machine for `ROS-I`, I would recommend something with these specs:

1. Dedicated ethernet port
2. 6 CPU cores (or 4 with Hyperthreading enabled) Basically, If you have an `I7-4th Gen` or `I5-8th gen` you should be good to go.
3. 16 Gigabytes of ram at 2400 KHz minimum (You can add ram later, but at least check that you can get that much) People can get by with less, but IDE's like that made by Jetbrains are very RAM intensive.
4. _Not_ a `4K` screen - 1920x1080p works great for these applications. Especially if you are on a laptop, the `4K` only makes things worse.
5. Ideally, use a desktop instead - desktops have dedicated NIC cards that are great with static IPs, especially dual-nic machines.

I should also mention that, if you plan on using `Matlab` on this machine as well, you will need to activate the `Matlab` license in the correct hardware configuration that you want to use with `ROS-I`. In other words, `Matlab` will complain if you unplug your USB to Ethernet adapter that you used to drive the robot arm. This is just a quirk of `Matlab` I thought you should be aware of. I have friends that use this repo alongside `Matlab` so I thought you should know. If this is an issue, there is an executable under the `Matlab` installation location for \*nix systems that allows you to re-license your `Matlab` machine such that it has the hardware configuration that you plan on using consistently.

## ROS Setup

First, you will need to make sure you have `ros-kinetic-desktop-full` installed on your ubuntu 1604 linux machine. My ROS catkin workspace is located here:
```
/home/$USER/catkin_ws/
```
This may be different on your system. If you have not yet done this installation you can go to [this link](https://wiki.ros.org/kinetic/Installation/Ubuntu).

I have some environment variables set up in my ~/.bashrc file.
This is the line that connects my path to ROS 1 setup.bash:
`source /opt/ros/kinetic/setup.bash`
I have to source this in my catkin workspace:
`source ~/catkin_ws/devel/setup.bash`
We also have to specifiy where `roscore` is running. We will be specifying that it runs on the Ubuntu laptop you are installing `ROS-I` and `MoveIt!` onto:
```
# Grab the machine IP addresses
machine_ip=(`hostname -I`)
rosport=':11311'
rosmasterbegin='http://'
# Now we set the ip location for roscore
export ROS_MASTER_URI=$rosmasterbegin${machine_ip[0]}$rosport
# This is the hostname of the current machine.
export ROS_HOSTNAME=${machine_ip[0]}
# This is the ROS distribution that we are running
export ROS_DISTRO=kinetic
# Sometimes you need to set ROS_IP for transforms and the parameter server
# https://answers.ros.org/question/163556/how-to-solve-couldnt-find-an-af_inet-address-for-problem/
export ROS_IP=${machine_ip[0]}
```
Supposedly you do not need to set ROS_MASTER_URI or ROS_HOSTNAME, but I seem to get more stable results setting these environment variables, I am telling you to do the same. Also, if you plan on using `Matlab` alongside this repository, you *will* need to set those environment variables in your `~/.bashrc` file.
If you do not want to use `hostname -I`, you can run `ifconfig` and look for `inet addr: xxx.xxx.xxx.xxx` to set the address.
Make sure to run `source ~/.bashrc` in all open terminals once that is all done. If you want an official explanation to what these variables are doing, [check out this link.](http://wiki.ros.org/ROS/EnvironmentVariables)

This is the [main link](http://wiki.ros.org/Industrial/Install) to install `ROS-Industrial`. Because we will be building from source, however, you can skip this link. I only show you this because some people do not like not seeing other options. I would not rely on this link to give you what you need.

You now need to git clone [this repository](https://github.com/ros-industrial/industrial_core) for the core functionalities of ros-industrial.

## MoveIt! Installation

Next, you will want to go to the [link provided here](http://docs.ros.org/kinetic/api/moveit_tutorials/html/doc/trac_ik/trac_ik_tutorial.html) and run the `sudo apt-get install xxxxxxx` commands.

You will also have to apt install this thing:

`sudo apt-get install ros-kinetic-tf2-geometry-msgs`

Now we need to install MoveIt!. I am instructing you to go to [this link here to install](http://moveit.ros.org/install/source/) from source. I have talked with many people on this subject, and they all recommend building from source, so I am telling you to as well. When you build from source, make sure that you go to the bottom of the page and use the `build from source` instructions that uses the `ROS_DISTRO` environment variable; but do not forget the `apt-get` commands in those instructions. Also, where the MoveIt advanced install instructions say to do `catkin build`, you really want to do `catkin_make`. __If you do not do this, your catkin workspace will be BROKEN.__ As a reminder, this is the [link to the main webpage](https://moveit.ros.org/) for the MoveIt libraries. This should not be necessary, but it is nice to know where other documentation is located.

For clarity, I have copied the instructions over from that link I provided:
```
cd cakin_ws
wstool init src
wstool merge -t src https://raw.githubusercontent.com/ros-planning/moveit/${ROS_DISTRO}-devel/moveit.rosinstall
wstool update -t src
rosdep install -y --from-paths src --ignore-src --rosdistro ${ROS_DISTRO}
catkin config --extend /opt/ros/${ROS_DISTRO} --cmake-args -DCMAKE_BUILD_TYPE=Release
catkin_make
```

## blahConfig.cmake missing

This error happens when `catkin_make` cannot find the package. As an example, in case you are missing `ros_industrial_core`, just git clone this `ROS-I` repo [into your catkin workspace](https://github.com/ros-industrial/industrial_core.git).

If there are any other packages missing, I suggest you google them. If you have a crap ton of missing packages, it is possible you messed up your installation of ROS Kinetic. The easiet solution for me is to reinsall ROS.

## ABB Specific Installation

Once you have built the moveit libraries from source, we need to get the ABB specific packages.

You need to `cd` into your catkin workspace source folder, which is for me

`~/catkin_ws/src/`

You will need to git clone some repositories into your workspace.

Git clone [this repo](https://github.com/ros-industrial/abb.git) on the `kinetic-devel` branch. This repo gives support for some of the robots you may need, but not all of them. 

Run `git branch -a` to figure out what branch you are currenly on. If you are on the wrong branch, run

`git checkout kinetic-devel`

that will put you on the correct branch.

The other `ABB` specific package is the `abb_experimental` package provided by `ROS-I`. Their version of the package includes the `ABB IRB1600 1.2` robot, but the robot we have is the `ABB IRB1600 1.45`. Therefore, in order to make the installation easier, you can git clone my fork instead, [which is linked](https://github.com/thatrobotguy/abb_experimental) here.

Now that you have downloaded all of those repositories into your workspace, you can finally run `catkin_make` in
`/home/$USER/catkin_ws/`.

I should note that you should NOT `catkin_make` until everything is git cloned, except for after using `wstool` to get the `MoveIt!` repositories when building from source. That compilation will take the longest, so reserve 15-20 minutes to do that.

## How to start up the ABB IRB 1600/120 with ROS

Now that you have ROS installed with the ABB packages, we have to now look at the robot arm itself and the teach pendant. The first order of business is to boot the robot arm into `System2_ROS`. Assuming you are currently booted into `System2` or `System1`, you will need to switch OSs. If you are brand spanking new to this, and you don't have the robot set up with the server socket program in `RAPID`, you can follow the instructions [here](https://wiki.ros.org/abb/Tutorials). That tutorial explains everything required to get the server ready to accept ROS commands through `MoveIt!`. Once the Teach Pendant is booted up to the correct operating system, click on the ABB logo in the upper left hand corner. Then click `Control Panel`. Then look down low at the last option in the list above the boot options. The option you want to click is `installed systems`. You then need to select the `System2_ROS` option and then look down and to the right and select `Activate`. Select `Yes` to start the reboot. The pendant will then reboot to the ROS arm configuration. Once that is done, you need to click on the “ABB” in the upper left and then select the `Program editor` option. Then you select `Debug` on the bottom and then select `PP to main` and then you should be set to go to…

STOP! NEVER FORGET THIS LAST STEP!

Press `play` on the teach pendant. ONLY THEN can you execute trajectories.

BUT WAIT! THE TEACH PENDANT ERRORS OUT!

TURN ON AUTOMATIC MODE. If you are in Manual mode, you must be pressing the motors enable grip down just enough to enable the motors before your program will run, even if you are controlling everthing with ROS. If you forget to do so, the teach pendant will throw errors. If you are a total nube, however, I suggest you use Manual mode to start until you know your program is good.

Now that you have installed everything, you can now run this command to start the simulated robot up with ROS:
```
roslaunch abb_irb1600_6_12_moveit_config moveit_planning_execution.launch robot_ip:=192.168.100.100 sim:=true
```
Or start the real robot with ROS:
```
roslaunch abb_irb1600_6_12_moveit_config moveit_planning_execution.launch robot_ip:=192.168.100.100 sim:=false
```
The only difference being the `sim:=true/false` at the end.

If you are using the real robot ABB IRB 120 3Kg, you use this command:
```
roslaunch abb_irb120_moveit_config moveit_planning_execution.launch robot_ip:=192.168.125.1 sim:=false
```
Remember, If you do not know the `robot_ip`, you can use `arp-scan` to find it.

## Example Robot Arm controller

There are 2 main middleware ROS nodes provided in this package. The first node starts up an example action server that receives a pose message and then handles the communication with the robot launch file. When you start the action server defined as the `abb_mover` executable, _right after you start the ABB robot node_, the action server will automatically set the orientation of any received poses to the rpy angles set in the referenced yaml file. All you need to do is send "x,y,z" values that are non-zero and the robot will handle the joint orientation. Currently, you need to have the robot set to the zero position before you start the action server. I highly recommend changing this so that you can arbitrarily move the robot (thorugh RViz) before you start the action server. This is the simpler, dumber way to drive the robot, as you have know control of the EoAT orientation with this node.

The other ROS node provides an interface to send the robot a generic X/Y/Z position and W/X/Y/Z quaternion orientation. This is the recommended node to use as it forces the user to listen to frame transforms like a good robotics engineer. I also gives more flexibility for the programmer to reach a valid pose if orientation does not matter.

It should be noted that this ros package is simply an example on how to control the arm with ROS 1. You should be able to get the arm up and running in a couple hours with a simple pose publisher _that publishes valid possible poses in the task space_. Do not worry: the MoveIt! IK planner is smart, but do remember, the collision for link 2, at the moment, is currently too forgiving (convex hulls again).

I have created a ROS node that acts as the middleware to ROS-I. Basically, you need to create a `move_group` message, not just a `geometry_msgs/Pose`, so I have written an abstract controller that wraps the `move_group` functionality such that you only need to publish a `geometry_msgs/Pose` to the `/do_arm_traj` topic.

This is the command to start the middleware ROS node:
```
roslaunch abb_1600_driver abb_full_pose.launch
```

If you don't already have a ROS node generating `geometry_msgs/Pose` messages to the correct ROS topic, but you still want to demo the ROS node and make sure it works, here is a CLI command that you can run for the ABB IRB1600 1.45 length 6 kg robot:
```
rostopic pub /do_arm_traj geometry_msgs/Pose "position:
  x: 0.510
  y: -0.510
  z: 0.750
orientation:
  x: -0.50
  y: 0.50
  z: -0.50
  w: 0.50"
```
I specify the larger robot simply because the X/Y/Z components of the EoAT are too far away for the smaller robot.

Once you start working on your own ROS node, you may want to check out my extra notes under `frame_files/findingframes.md` for more info on `tf` frames. I suggest you figure out valid frames BEFORE you start sending them to the real robot (aka set `sim:=true` first). You will need the frame transforms when you want to localize the robot into your custom workspace.

## Final notes

I find that the ROS stuff is more stable if you start the robot `pp to main` first, then run the launch file. You also do not need to start roscore separately as the launch file will do that for you. If you run into issues with roscore not working, you probably need to make sure you set the environment variables properly.

I believe that this is everything required to get the arm to work. You should call the uppermost launch file described earlier in their own custom launch file in their own custom ROS package so that they do not have any of the ROS-Industrial packages in their personal git repositories. I have seen teams make this mistake already.
## Possible Errors

You may get an error when you start the robot launch file. You may get something like this:
```
[ERROR] [1571681258.303701135]: The kinematics plugin (manipulator) failed to load. Error: According to the loaded plugin descriptions the class trac_ik_kinematics_plugin/TRAC_IKKinematicsPlugin with base class type kinematics::KinematicsBase does not exist. Declared types are  abb_irb2400_manipulator_kinematics/IKFastKinematicsPlugin cached_ik_kinematics_plugin/CachedKDLKinematicsPlugin cached_ik_kinematics_plugin/CachedSrvKinematicsPlugin kdl_kinematics_plugin/KDLKinematicsPlugin lma_kinematics_plugin/LMAKinematicsPlugin srv_kinematics_plugin/SrvKinematicsPlugin
[ERROR] [1571681258.303752400]: Kinematics solver could not be instantiated for joint group manipulator.
```
This is solved by installing `trac-ik` like this:
```
sudo apt install ros-kinetic-trac-ik
```

Sometimes you may get this error while running in manual mode:
```
[ERROR] [1544201172.958564157]: Controller is taking too long to execute trajectory (the expected upper bound for the trajectory execution was 4.625994 seconds). Stopping trajectory.
```
This is why I forked the original `abb_experimental` repository. The stock version is kind of jank. I mentioned in the `README.md` file of that [forked repo](https://github.com/thatrobotguy/abb_experimental) that there are 3 launch files that were edited (not including the robot model changes). You need to look at that 3rd launch flie I provided and give the robot more time to execute trajectories.
You can do that by changing this line:
```
<param name="trajectory_execution/execution_duration_monitoring" value="true" />
```
To this line:
```
<param name="trajectory_execution/execution_duration_monitoring" value="false" />
```
It will no longer care about how long the trajectories take. When you start using Automatic mode, you should probably turn this back on.
You could also make this value larger:
```
<param name="trajectory_execution/allowed_goal_duration_margin" value="3.5"/> <!-- default 0.5 -->
```
But if you are in manual mode it is easier to just turn off this safety. [This link](https://answers.ros.org/question/196586/how-do-i-disable-execution_duration_monitoring/) has more discussions about this problem.

You may also encounter [this issue](https://github.com/qboticslabs/mastering_ros/issues/24) while trying to drive the robot. Also, if you are wanting to listen to transforms, you need to create a listener AFTER you call `initnode(argc, argv, "mynodename")`. This means that you cannot create a global variable listener outside of `main()` or `rosrun mypackage mynodewithtflisteners` will not work.

## Updates

I have decided that, given the amount of changes to the `abb_experimental` ros package, that it is best to simply fork it into my own github account and then modify everything there. Not only are the 3 launch files changed, but the .urdf and .xacro have also been updated with the correct link 2 lengths. Link 2 is actually 0.7 meters long, not 0.485 meters long. The original ROS package meshes describe the 1.2 meter robot, not the 1.45 meter robot that WPI owns (both are the 6kg version though). Simply clone [this repository](https://github.com/thatrobotguy/abb_experimental) instead of the stock `abb_experimental` repository and the edits will be taken care of for you. __Do note, however, that colision protection is not as effective as it would be normally__. What I mean is that the current collision mesh file for link 2 is currently the same as the visual file. It is better to use the convex hull of the visual file to allow for some wiggle room. Ideally, the ABB IRB1600 1.45 would be in a package called `abb_irb1600_6_145_moveit_config`, but I have not gotten that far yet. I have simply been reusing the `abb_irb1600_6_12_moveit_config` ros package. Once this is done, I can merge with the original repository hosted by [these guys](https://github.com/ros-industrial).

## Liability

I can take no responsibility for any actions by any entity through the use of any code or documentation or any other files provided in this repository. This software interfaces with big, large, dangerous robots that can kill you if you aren't paying attention, so use at your own risk.

#### More good links:
```
https://docs.ros.org/kinetic/api/moveit_tutorials/html/doc/move_group_interface/move_group_interface_tutorial.html
https://docs.ros.org/kinetic/api/moveit_tutorials/html/index.html
https://wiki.ros.org/ROS/EnvironmentVariables
https://industrial-training-master.readthedocs.io/en/latest/_source/session4/Motion-Planning-CPP.html#further-information-and-resources
https://industrial-training-master.readthedocs.io/en/latest/_source/session2/Launch-Files.html
https://industrial-training-master.readthedocs.io/en/latest/_source/session2/Actions.html
https://industrial-training-master.readthedocs.io/en/latest/_source/session3/Build-a-Moveit!-Package.html
http://wiki.ros.org/abb/Tutorials
https://erlerobotics.gitbooks.io/erlerobot/en/ros/tutorials/rosnavigating.html
```
## Documentation written by thatrobotguy
