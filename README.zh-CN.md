# LIFT3 SDK

[English](README.md) | **简体中文**

LIFT3 由移动平台和 X5 机械臂组成，本文档只针对**移动平台**。X5 部分请参考 [ARX5 SDK 文档](https://github.com/ARXroboticsX/ARX5_beta/blob/main/readme.md)。

> **⚠️ 安全须知**
>
> - 使用机器人时务必确保安装稳定，以基座为轴心两米半径内保持空旷，当心碰撞易碎物品及造成人员损伤。
> - 出现紧急情况请先关闭电源。

## 目录

- [环境要求](#环境要求)
- [硬件连接](#硬件连接)
- [紧急情况处理](#紧急情况处理)
- [CAN 配置](#can-配置)
- [机身 SDK](#机身-sdk)
- [遥控器控制](#遥控器控制)

## 环境要求

- Ubuntu 24.04
- Python 3.12（Ubuntu 24.04 自带版本；`liftlib` 为编译好的模块，只支持该版本）
- 环境配置与 X5 相同，见 [ARX5 SDK 文档](https://github.com/ARXroboticsX/ARX5_beta/blob/main/readme.md)

## 硬件连接

### 线材连接

- **无需 GPU**：使用机载 NUC 进行控制。
- **需要 GPU**：将传感器及信号线接入自己的 PC。

两种情况连接方式相同，如下所示：

<p align="center">
  <img src="images/wiring.jpg" width="720" alt="线材连接：左臂、底盘、右臂与 CAN USB">
</p>

将 **left arm**、**right arm**、**chassis** 三部分连接至机器人。连接后 CAN 信号会在内部汇总，只需将 CAN 的 USB 线接到电脑 USB 口即可。

> **注意：** CAN 的 USB 线要单独占用一个 USB 口，不能与其他设备共用，防止信号干扰。

### 电源

> **注意：** 默认不配备电池组，仅支持有线方式供电。

<p align="center">
  <img src="images/power-adapter.jpg" width="480" alt="电源适配器有线供电">
</p>

### 电池组使用方式

本设备采用双电池组，支持热插拔快速更换。

电池启动方式：点按一下电源键，再长按，等四个指示灯全部亮起即通电。

<p align="center">
  <img src="images/battery-power-on.jpg" width="480" alt="电池通电，四个指示灯全亮">
</p>

电池拆装：

<p align="center">
  <img src="images/battery-swap.png" width="600" alt="电池拆装">
</p>

充电管家：

<p align="center">
  <img src="images/battery-charger.png" width="460" alt="电池充电管家">
</p>

## 紧急情况处理

- 启动前先将急停开关**旋开**。
- 出现紧急情况时，用脚或工具按下急停开关。**注意：断电时升降平台会下坠。**

<p align="center">
  <img src="images/emergency-stop.jpg" width="420" alt="急停开关">
</p>

## CAN 配置

双击 `LIFT-SDK/ARX_CAN_doctor` 启动。与单臂设置方法相同：配置时只连接目标设备。

| 设备 | CAN |
|---|---|
| 左臂 | `can1` |
| 左主臂 | `can2` |
| 右臂 | `can3` |
| 右主臂 | `can4` |
| 底盘 | `can5` |
| 遥控器 | `can6` |

1. **如使用遥控器，先单独连接遥控器**，依次配置为 2、4、6。只有遥控器的配置与单臂/底盘不同：遥控器会一次性输出 3 路 CAN。

   <p align="center">
     <img src="images/can-config.png" width="600" alt="在 ARX_CAN_doctor 中配置遥控器的 3 路 CAN">
   </p>

2. 然后**逐个**插入其余设备，进行对应 ID 配置。配置完一个后可以不拔，直接插入下一个继续设置——逐个插入只是为了区分对应设备。
3. 设备如未更换，只需配置一次。

## 机身 SDK

### 参考案例

在 `LIFT-SDK/` 目录下运行：

```bash
python3 demo_sdk.py
```

默认使用 `can5`，机器人类型选择 `LIFT2026_18`：

```python
CAN_ROBOT = 'can5'  # 机器人CAN接口
ROBOT_TYPE = RobotType.LIFT2026_18  # 带头部的 X7 用 X72026; 无头部的 LIFT: LIFT2026_18
```

> 控制循环中需持续调用 `robot.loop()`（约 400 Hz），写法见 `demo_sdk.py`；超过 0.5 s 未调用会自动刹车。

### 1. 高度控制

```python
robot.set_height(set_lift_height)  # 目标高度 (m)，范围 0–0.38
```

### 2. 底盘整体速度控制（模式 1）

```python
robot.set_chassis_cmd(x, y, z, 1)  # x、y: m/s；z: rad/s；mode = 1
```

### 3. 轮速控制（模式 3）

```python
robot.set_chassis_cmd(0, 0, 0, 3)    # 切换到模式 3
robot.set_wheel_vel(v1, v2, v3, v4)  # 四轮转速 (rad/s)，轮子 ID 见下图
```

<p align="center">
  <img src="images/wheel-ids.png" width="260" alt="底盘轮子 ID">
</p>

### 4. 头部控制

角度单位 rad。**仅 X7 机型**，LIFT 机型调用为空操作。

```python
robot.set_head_yaw(0.2)
robot.set_head_pitch(-0.2)
```

### 5. 状态获取

```python
print(f"当前高度: {robot.get_height()}")
print(f"轮速: {robot.get_wheel_vel()}")
print(f"头部角度: {robot.get_head_pos()}")  # [yaw, pitch]
```

## 遥控器控制

### 硬件连接

1. 将两条主臂（leader arm）的接口插入遥控器。

   <p align="center">
     <img src="images/remote-leader-arms.jpg" width="560" alt="两条主臂接入遥控器">
   </p>

2. 用自带的 G 型夹将主臂固定在桌面。

   <p align="center">
     <img src="images/remote-clamp.jpg" width="420" alt="用 G 型夹固定主臂">
   </p>

3. 用附带的 USB 线将遥控器连接至电脑。

   <p align="center">
     <img src="images/remote-usb.jpg" width="420" alt="遥控器 USB 连接">
   </p>

### 遥控器使用案例

在 `LIFT-SDK/` 目录下运行：

```bash
python3 demo_joy.py
```

- **左摇杆**：上下控制升降高度，左右控制绕 Z 轴旋转
- **右摇杆**：控制 x、y 方向移动

<p align="center">
  <img src="images/joystick-axes.png" width="520" alt="遥控器摇杆方向">
</p>
