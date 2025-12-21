让我通过经典的"海龟模拟器"（turtlesim）来详细介绍ROS2的核心概念：节点、话题、服务、动作等。

## 🎯 ROS2 turtlesim 示例环境

首先启动基础环境：

# 终端1: 启动turtlesim模拟器
ros2 run turtlesim turtlesim_node

# 终端2: 启动海龟控制节点
ros2 run turtlesim turtle_teleop_key

现在让我们分层解析：

## 1. 节点（Nodes）

### 什么是节点？
节点是ROS2中的基本执行单元，每个节点负责完成单一、模块化的功能。

### turtlesim中的节点：
# 查看当前运行的节点
ros2 node list
输出：
/turtlesim
/teleop_turtle

### 节点详细信息：
# 查看节点信息
ros2 node info /turtlesim
ros2 node info /teleop_turtle

### 节点输出示例：
/turtlesim
  Subscribers:
    /turtle1/cmd_vel: geometry_msgs/msg/Twist
  Publishers:
    /turtle1/color_sensor: turtlesim/msg/Color
    /turtle1/pose: turtlesim/msg/Pose
  Service Servers:
    /turtle1/teleport_absolute: turtlesim/srv/TeleportAbsolute
    /turtle1/teleport_relative: turtlesim/srv/TeleportRelative
    /clear: std_srvs/srv/Empty
    /kill: turtlesim/srv/Kill
    /reset: std_srvs/srv/Empty
    /spawn: turtlesim/srv/Spawn
    /turtlesim/describe_parameters: rcl_interfaces/srv/DescribeParameters
    ...
  Service Clients:
    ...
  Action Servers:
    /turtle1/rotate_absolute: turtlesim/action/RotateAbsolute
  Action Clients:

## 2. 话题（Topics）

### 什么是话题？
话题是节点间通信的**发布-订阅（Pub-Sub）** 通道，用于**异步、单向、流式**数据传输。

### turtlesim中的话题：
# 查看所有话题
ros2 topic list

# 查看话题详细信息
ros2 topic info /turtle1/cmd_vel

### 主要话题：
| 话题 | 类型 | 发布者 | 订阅者 | 作用 |
|------|------|--------|--------|------|
| /turtle1/cmd_vel | geometry_msgs/Twist | teleop_turtle | turtlesim | 控制海龟运动 |
| /turtle1/pose | turtlesim/Pose | turtlesim | - | 发布海龟位置姿态 |
| /turtle1/color_sensor | turtlesim/Color | turtlesim | - | 发布海龟底部颜色 |

### 话题通信示例：
# 监听海龟位置
ros2 topic echo /turtle1/pose

# 查看话题数据
ros2 topic hz /turtle1/pose

# 手动发布控制命令
ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 2.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 1.8}}"

## 3. 服务（Services）

### 什么是服务？
服务是**请求-响应**模式的同步通信机制，用于**一次性的、需要确认的**操作。

### turtlesim中的服务：
# 查看所有服务
ros2 service list

# 查看服务类型
ros2 service type /spawn

### 主要服务：
| 服务 | 类型 | 作用 |
|------|------|------|
| /spawn | turtlesim/Spawn | 生成新海龟 |
| /kill | turtlesim/Kill | 杀死海龟 |
| /clear | std_srvs/Empty | 清除轨迹 |
| /reset | std_srvs/Empty | 重置模拟器 |
| /turtle1/teleport_absolute | turtlesim/TeleportAbsolute | 绝对位置传送 |
| /turtle1/teleport_relative | turtlesim/TeleportRelative | 相对位置传送 |

### 服务调用示例：
# 生成新海龟
ros2 service call /spawn turtlesim/srv/Spawn \
  "{x: 5.5, y: 5.5, theta: 0.0, name: 'turtle2'}"

# 杀死海龟
ros2 service call /kill turtlesim/srv/Kill "{name: 'turtle2'}"

# 传送到指定位置
ros2 service call /turtle1/teleport_absolute turtlesim/srv/TeleportAbsolute \
  "{x: 1.0, y: 1.0, theta: 0.0}"

## 4. 动作（Actions）

### 什么是动作？
动作是**长期运行、可取消、有反馈**的任务执行机制，结合了话题和服务的优点。

### turtlesim中的动作：
# 查看所有动作
ros2 action list

# 查看动作类型
ros2 action info /turtle1/rotate_absolute

### 动作通信流程：
Action Client → Goal Request → Action Server
             ← Feedback ←
             ← Result ←

### 动作示例：
# 发送旋转目标（转到90度方向）
ros2 action send_goal /turtle1/rotate_absolute \
  turtlesim/action/RotateAbsolute "{theta: 1.57}"

# 带反馈查看
ros2 action send_goal /turtle1/rotate_absolute \
  turtlesim/action/RotateAbsolute "{theta: 3.14}" \
  --feedback

## 5. 参数（Parameters）

### 什么是参数？
参数是节点的**配置变量**，可在运行时动态修改。

### turtlesim中的参数：
# 查看参数
ros2 param list

# 获取参数值
ros2 param get /turtlesim background_r

# 设置参数（改变背景颜色）
ros2 param set /turtlesim background_r 150
ros2 param set /turtlesim background_g 100
ros2 param set /turtlesim background_b 200

# 保存和加载参数
ros2 param dump /turtlesim > turtlesim_params.yaml
ros2 param load /turtlesim turtlesim_params.yaml

## 📊 ROS2 通信架构总结

### 层级关系图示：
┌─────────────────────────────────────────────────────────┐
│                    ROS2 应用程序                         │
├─────────────────────────────────────────────────────────┤
│  节点(Node)  ──── 节点(Node)  ──── 节点(Node)            │
│     │              │              │                     │
│   话题(Topic)    服务(Service)   动作(Action)           │
│     │              │              │                     │
│  消息(Message) 请求/响应(Request/Response) 目标/反馈/结果│
└─────────────────────────────────────────────────────────┘

### 通信方式对比：




| 特性 | 话题(Topic) | 服务(Service) | 动作(Action) |
|------|-------------|---------------|--------------|
| 通信模式 | 发布-订阅 | 请求-响应 | 目标-反馈-结果 |
| 同步性 | 异步 | 同步 | 异步（有同步部分） |
| 数据流 | 单向流式 | 一次性双向 | 长期双向 |
| 实时性 | 实时推送 | 等待响应 | 持续反馈 |
| 适用场景 | 传感器数据 | 开关控制 | 导航任务 |
| 示例 | 速度命令 | 生成海龟 | 旋转到角度 |

## 🔧 实用命令总结

# 系统监控
ros2 node list                 # 列出节点
ros2 topic list                # 列出话题
ros2 service list              # 列出服务
ros2 action list               # 列出动作
ros2 param list                # 列出参数

# 详细信息
ros2 node info <node_name>
ros2 topic info <topic_name>
ros2 interface show <msg_type>

# 实时监控
ros2 topic echo <topic_name>
ros2 topic hz <topic_name>
ros2 topic bw <topic_name>

# 手动交互
ros2 topic pub <topic> <type> <data>
ros2 service call <service> <type> <data>
ros2 action send_goal <action> <type> <data>

## 🎯 turtlesim 完整演示脚本

#!/bin/bash
# turtlesim_demo.sh

echo "1. 启动turtlesim..."
ros2 run turtlesim turtlesim_node &
sleep 2

echo "2. 查看节点..."
ros2 node list

echo "3. 查看话题..."
ros2 topic list

echo "4. 移动海龟..."
ros2 topic pub /turtle1/cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 1.0, y: 0.0, z: 0.0}, angular: {x: 0.0, y: 0.0, z: 0.5}}" \
  --once

echo "5. 生成第二只海龟..."
ros2 service call /spawn turtlesim/srv/Spawn \
  "{x: 2.0, y: 2.0, theta: 1.57, name: 'leo'}"

echo "6. 设置背景颜色..."
ros2 param set /turtlesim background_r 100
ros2 param set /turtlesim background_g 150
ros2 param set /turtlesim background_b 200

echo "7. 执行旋转动作..."
ros2 action send_goal /turtle1/rotate_absolute \
  turtlesim/action/RotateAbsolute "{theta: 3.14}"

这个示例完整展示了ROS2的核心通信机制。建议您实际操作一遍，理解会更加深刻！