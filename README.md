# Ridgeback的使用


### 一.跟ridgeback通讯上:
(1)在自己电脑创建有线连接

(2)通过ssh连接上移动平台

(3)更改.bashrc
>export ROS_MASTER_URI=http://192.168.131.1:11311   #ridegback的IP地址
>
export ROS_IP=192.168.31.50     #你电脑的IP地址

使环境生效,在终端输入:
```
source .bashrc
```
(4)打开终端,修改`/etc/hosts`文件
```
sudo gedit /etc/hosts
```
加入下面代码:
>192.168.131.1 CPR-R100-0038     #创建有线连接移动平台的IP 名字
>
192.168.131.50 GJXS     #创建有线连接你电脑的IP 名字

(5)查询话题列表
```
rostopic list
```
可以看到移动平台的话题
(6)控制移动平台移动:
```
rostopic pub -r 0.5 /cmd_vel geometry_msgs/Twist '{linear: {x: 0.00, y: 0, z: 0}, angular: {x: 0, y: 0, z: 0.0}}' 
```
可以控制移动平台的运动,到这里跟运动平台的通讯就完成

### 二.启动ridgeback_viz仿真界面
(1)导入模型
```
roslaunch ridgeback_viz view_robot.launch
```
新开终端
```
roslaunch ridgeback_viz view_model.launch
```

(2)显示激光雷达和imu
>添加激光雷达(LaserScan)和imu
>
激光雷达(LaserScan)的topic设置为:/front/scan
>
imu的topic设置为:/imu/data_raw

(3)查看电机运行情况
```
rosrun rqt_robot_monitor rqt_robot_monitor
```

### 三.利用激光雷达导航
1.没有地图的用法:

在导航演示中，Ridgeback试图在用户指定的容差范围内达到世界坐标系的特定目标。由move_base生成的2D导航从odometry，激光扫描仪和目标姿态中获取信息，并输出安全速度命令。在这个演示中，move_base的配置被设置为在odometric框架中没有地图的导航（即不参考地图）

(1)要启动导航演示，请运行：
```
roslaunch ridgeback_navigation odom_navigation_demo.launch
```

(2)使用建议的rviz配置启动进行可视化：
```
roslaunch ridgeback_viz view_robot.launch mode:=navigation
```
要将目标发送到机器人，请从顶部工具栏中选择2D导航目标工具，然后单击rviz视图中的任意位置以设置位置。或者点击并稍微拖动来设置目标位置和方向。

2.用已知地图导航的用法:

使用amcl，Ridgeback能够在一张已知的地图中进行全球本地化。AMCL从odometry，激光扫描仪和现有地图中获取信息，并估算机器人的姿势。

(1)开始AMCL演示：
```
roslaunch ridgeback_navigation amcl_demo.launch map_file:=/path/to/my/map.yaml
```
如果您不指定map_file，则默认为Ridgeback模拟器生成的缺省“Ridgeback Race”环境的预先制作的映射。如果您在自己的环境中使用真正的Ridgeback，则一定要用使用gmapping演示创建的地图覆盖此项。

(2)在导航之前，您需要通过在地图中设置机器人的姿势来初始化本地化系统。这可以通过在rviz中使用2D姿态估计或通过设置amcl initial_pose参数来完成。使用建议的rviz配置启动进行可视化：
```
roslaunch ridgeback_viz view_robot.launch mode:=localization
```

3.使用Gmapping制作地图

(1)在这个演示中，Ridgeback使用gmapping生成地图。首先启动gmapping演示启动文件（最好在机器人上）：
```
roslaunch ridgeback_navigation gmapping_demo.launch
```
(2)在工作站上，使用建议的配置启动rviz：
```
roslaunch ridgeback_viz view_robot.launch mode:=gmapping
```
你必须慢慢驾驶Ridgeback来制作地图。随着激光扫描仪出现障碍，它们将被添加到地图中，该图显示在rviz中。您既可以使用交互式标记进行手动驱动，也可以通过发送导航目标（如上所述）进行半自主驱动。

(3)使用map_saver保存制作的地图：
```
rosrun map_server map_saver -f mymap
```
这将创建一个mymap.yaml与mymap.pgm您的当前目录中的文件。

### 四.IMU传感器的使用
(1)IMU校准

Ridgeback包含一个IMU，可以用它来估计自己在太空中的方向。这通过使用三个传感器的组合来实现：陀螺仪，加速度计和磁力计。这些措施分别是：角运动，加速度和磁场。对角运动进行积分以实现相对估计，然后将其锚定到两个绝对参考点：由加速度计提供的重力矢量以及由磁力计提供的磁场矢量。尽管向下指向的引力是相当一致的，但是磁参考矢量由于偏角而具有偏移，并且也很容易受到干扰。Ridgeback以外的干扰很少（例如，在靠近强磁场的地方驾驶），但通过校准可以减少电池，电机，钣金零件等板上Ridgeback磁场的影响。

校准Ridgeback的磁力仪是一个简单的过程。为获得最佳性能，建议在季节性基础上执行此操作。

校准开始:
```
rosrun ridgeback_base calibrate_compass
```
Ridgeback应该慢慢旋转60秒，记录磁力计数据。

完成后，系统将提示您输入用户密码以保存新的校准`$ROS_ETC_DIR`。完成后，重新启动ROS服务以开始使用新校准：
```
sudo service ridgeback restart
```
(2)启动模拟器
要加载模拟的Ridgeback，请运行以下命令：
```
roslaunch ridgeback_gazebo ridgeback_world.launch
```