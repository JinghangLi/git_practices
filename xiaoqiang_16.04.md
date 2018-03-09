## 写在前面
在将小强机器人的系统由14.04升级到16.04时，我主要遇到了以下几个问题。
1. ssh连接时出现
```
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
```
[解决该报错方法](http://blog.csdn.net/liuhang03/article/details/50323401)

2. 小强机器人的 **hostname** 需是 **xiaoqiang-desktop** `/etc/hosts` 文件中也要注意为
```
127.0.0.1	localhost
127.0.1.1	xiaoqiang-desktop
```

3. 在使用 ***Rviz*** 接收图像信息时，如果按照教程在 **open config** 对话框中找不到小强机器人对应的文件，则可以采用将相对应的 ***config文件*** 拷贝到遥控端电脑中打开的方式.
4. 若想要在 **Rviz** 中 ***显示小强机器人的模型示意图*** 则需要在打开Rviz前 source 遥控端电脑安装的小强机器人模型包的 setup.sh 文件
```
$ source ~/Documents/ros/devel/setup.sh
$ export ROS_MASTER_URI=http://xiaoqiang-desktop:11311
$ rviz
```

## 教程正文
#### 几点说明
1. 电脑终端为两台
* 一台为小强机器人上搭载的电脑终端
* 一台为操作者使用的安装有 **Ubuntu 16.04 与 ROS kinetic** 的电脑作为遥控端，用于显示小强机器人发送的数据与向小强机器人发布命令，下文以 ***遥控端*** 代称.

2. *配置网络 机器人组装 软件说明 Ubuntu系统配置 开源仓库使用* 请详见 **用户手册**.
#### 教程1 准备
1. 遥控端电脑通过 *ssh* 与小强机器人连接

```
$ ssh xiaoqiang@192.168.1.101
```
* 启动遥控程序
```
$ rosrun nav_test control.py
```
可以通过方向键来控制小强的移动了。空格键是停止。Ctrl + C 退出程序
* 在遥控端系统的 `/etc/hosts` 文件中添加小强的ip
```
192.168.1.101 xiaoqiang-desktop
```
* 在小强终端中添加遥控端的ip地址
```
$ ssh -X xiaoqiang@192.168.1.101
$ sudo gedit /etc/hosts
```
输入WiFi分配给遥控端的IP地址（IP地址可在网络设置的连接信息中查到）
eg：192.168.1.xxx （遥控端计算机名）
* 视频传输
在遥控端打开一个终端输入
```
$ export ROS_MASTER_URI=http://xiaoqiang-desktop:11311
$ rosrun image_view image_view image:=/camera_node/image_raw
_image_transport:=compressed
```
如果一切正常就能够看到当前小强的摄像头画面了

#### 教程4 惯性导航自主测试
1. 在遥控端开一个终端，输入以下命令
```
$ ssh -X xiaoqiang@192.168.1.101
$ roslaunch nav_test fake_move_base_blank_map.launch
```

2. 安装小强机器人在Rviz中显示的模型软件包
```
$ mkdir ~/Documents/ros/src
$ cd ~/Documents/ros/src
$ catkin_init_workspace
$ git clone https://github.com/BlueWhaleRobot/xiaoqiang_udrf.git
#切换到 master 分支
$ cd xiaoqiang_udrf
$ git checkout master
#编译完成安装
$ cd ..
$ cd ..
$ catkin_make
```

3. 打开Rviz界面
```
$ source ~/Documents/ros/devel/setup.sh
$ export ROS_MASTER_URI=http://xiaoqiang-desktop:11311
$ rviz
```
当窗口打开后,点击左上角的 file->open,选择小强里的
/home/xiaoqiang/Documents/ros/src/nav_test/config/nav.rviz 文件
***若无法访问小强机器人的工作目录，则可以选择将配置文件复制到遥控端电脑***

4. 在小强终端的远程ssh窗口内输入命令
```
rosrun nav_test square.py
```

#### 教程9 使用 rostopic 控制 kinect 的俯仰角度
1. 在本地虚拟机新开一个窗口,启动 freenect_stack 驱动
ssh 登入小车主机
```
$ ssh -X xiaoqiang@192.168.1.101
$ roslaunch freenect_launch freenect-xyz.launch
```
**如果出现红色错误(驱动缺陷),请 ctrl+c 关闭命令后等待 6 秒(真的需要 6 秒),再次尝试允许上面的 roslaunch 命令。**

2. 发布电机角度控制命令
```
$ ssh -X xiaoqiang@192.168.1.101
$ rostopic pub /set_tilt_degree std_msgs/Int16 '{data: -20}' -r 1
```

#### 教程10 使用 kinect 进行自主移动避障
1. 在遥控端开启三个三个终端窗口，并分别 ssh 登入小车
* 第一个窗口
```
$ ssh -X xiaoqiang@192.168.1.101
$ roslaunch freenect_launch kinect-xyz.launch
```
* 第二个窗口
```
$ ssh -X xiaoqiang@192.168.1.101
$ rostopic pub /set_tilt_degree std_msgs/Int16 '{data: -19}' -r 1
```
* 第三个窗口
```
$ ssh -X xiaoqiang@192.168.1.101
$ roslaunch nav_test fake_move_base_blank_map.launch
```

2. 启动 Rviz
* 打开一个遥控端窗口
```
$ source ~/Documents/ros/devel/setup.sh
$ export ROS_MASTER_URI=http://xiaoqiang-desktop:11311
$ rviz
```
* 打开配置文件
点 击 RVIZ 界 面 左 上 角 的 OPEN CONFIG , 选 择 小 车 主 机 上 的
`/HOME/XIAOQIANG/DOCUMENTS/ROS/SRC/NAV_TEST/CONFIG/NAV_ADDWA_KINECT.RVIZ` 配置文件

3. 此时可在 Rviz 界面中发布2维导航点

###  教程14 在 gmapping 下使用激光雷达 rplidar a2 进行建图
1. 启动 gmapping 节点
```
$ ssh -X xiaoqiang@192.168.1.101
$ roslaunch gmapping slam_gmapping_xiaoqiang_rplidar_a2.launch
```
2. 打开 Rviz
```
$ source ~/Documents/ros/devel/setup.sh
$ export ROS_MASTER_URI=http://xiaoqiang-desktop:11311
$ rviz
```
打开小强 ros 工作目录下的`slam_gmapping/gmapping/launch/rplidar_a2_test.rviz` 配置文件

3. 遥控小强机器人开始建图
* 遥控端遥控小强机器人建图
```
$ ssh -X xiaoqiang@192.168.1.101
$ rosrun nav_test control.py
```
* 安卓手机遥控小强机器人建图
[app 下载链接](http://community.bwbot.org/uploads/files/1487658064497-xiaoqiang-with-control.apk)

4. 保存地图
ssh 登录小强,在小强 home 目录下保存为 work0 开头的文件
```
$ ssh -X xiaoqiang@192.168.1.101
$ rosrun map_server map_saver -f work0
```

### 教程15 AMCL 导航测试
1. 启动导航节点
```
$ ssh -X xiaoqiang@192.168.1.101
$ roslaunch nav_test xiaoqiang_a2_demo_amcl.launch
```
2. 启动 Rviz
```
$ source ~/Documents/ros/devel/setup.sh
$ export ROS_MASTER_URI=http://xiaoqiang-desktop:11311
$ rviz
```
打开小强 ROS 工作目录下的`NAV_TEST/CONFIG/XIAOQIANG_AMCL.RVIZ` 配置文件

***注意 在演示前请关闭 Odometry 下的 Covariance 防止卡顿***

### 教程17 利用 ORB_SLAM2 建立环境三维模型
1. 启动 ORB_SLAM2
打开遥控端的终端，输入以下命令
```
$ ssh -X xiaoqiang@192.168.1.101
$ roslaunch orb_slam2 start.launch
```
2. 开始建立环境三维模型
使用 Windows 客户端操作

3. 使用 rviz 查看建图效果
```
$ source ~/Documents/ros/devel/setup.sh
$ export ROS_MASTER_URI=http://xiaoqiang-desktop:11311
$ rviz
```
打开 `/home/xiaoqiang/Documents/ros/src/ORB_SLAM/Data/rivz.rviz` 配置文件，查看建图效果

4. 保存地图
```
$ ssh -X xiaoqiang@192.168.1.101
$ rostopic echo /map_save
```
地图文件会被保存进用户主目录的 `slamdb` 文件夹内

5. 地图的载入
将 `/home/xiaoqiang/Documents/ros/workspace/src/ORB_SLAM2/Examples/ROS/orb_slam2/Data` 文
件夹内的 `setting.yaml` 文件中的 `LoadMap` 的值设置为 1

### 教程18 利用 DSO_SLAM 建立环境三维模型
1. 直接运行(已安装dso)
```
$ rosrun dso_ros dso_live image: = /camera_node/image _raw calib= /home/xiaoqiang/Documents/ros/src/dso _ros/camera.txt mode= 1
```
