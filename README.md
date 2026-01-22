# Kinematics (Modified)

A modified Arduino Kinematics library based on [linorobot/kinematics](https://github.com/linorobot/kinematics), for differential drive (2WD, 4WD) and mecanum drive robots.

基于 [linorobot/kinematics](https://github.com/linorobot/kinematics) 库修改的 Arduino 运动学库，适用于差速驱动（2WD、4WD）和麦克纳姆轮机器人。

---

## Acknowledgments / 致谢

This library is based on the original kinematics library by **Juan Jimeno**.  
本库基于 **Juan Jimeno** 的原始 kinematics 库开发。

- Original Project / 原始项目：https://github.com/linorobot/kinematics  
- Reference / 参考文献：http://research.ijcaonline.org/volume113/number3/pxc3901586.pdf

---

## Modifications / 修改内容

### Fix: Inconsistent Rotation Radius Calculation / 修复：旋转半径计算不一致问题

**Problem / 问题描述：**  

In the original library, the rotation radius used in forward kinematics (`getRPM`) and inverse kinematics (`getVelocities`) were inconsistent:

原始库中，正运动学（`getRPM`）和逆运动学（`getVelocities`）使用的旋转半径定义不一致：

- Forward kinematics / 正运动学：`tangential_vel_ = angular_vel_z_mins_ * lr_wheels_dist_`
- Inverse kinematics / 逆运动学：`vel.angular_z = ... / ((fr_wheels_dist_ / 2) + (lr_wheels_dist_ / 2))`

**Solution / 修复方案：**  

Unified rotation radius to `(lr_wheels_dist_ + fr_wheels_dist_) / 2`, ensuring forward and inverse kinematics are consistent.

统一使用 `(lr_wheels_dist_ + fr_wheels_dist_) / 2` 作为旋转半径，确保正/逆运动学计算互逆。

```cpp
// Modified / 修改后
tangential_vel_ = angular_vel_z_mins_ * ((lr_wheels_dist_ + fr_wheels_dist_) / 2);
```

---

## Usage / 使用说明

The library requires the following robot specifications as input:

本库需要以下机器人参数作为输入：

- Motor maximum RPM / 电机最大 RPM
- Wheel diameter / 轮子直径
- Front-rear wheel distance (FR_WHEEL_DISTANCE) / 前后轮距
- Left-right wheel distance (LR_WHEEL_DISTANCE) / 左右轮距

---

## Functions / 函数接口

### 1. Constructor / 构造函数
```cpp
Kinematics(int motor_max_rpm, float wheel_diameter, float fr_wheels_dist, float lr_wheels_dist, int pwm_bits)
```
| Parameter | Description (EN) | 描述 (CN) |
|-----------|------------------|-----------|
| `motor_max_rpm` | Maximum motor RPM | 电机最大转速 |
| `wheel_diameter` | Wheel diameter in meters | 轮子直径（米） |
| `fr_wheels_dist` | Front-rear wheel distance in meters | 前后轮距（米） |
| `lr_wheels_dist` | Left-right wheel distance in meters | 左右轮距（米） |
| `pwm_bits` | PWM resolution (default 8 bits for Arduino Uno/Mega) | PWM 分辨率（Arduino Uno/Mega 默认 8 位） |

### 2. getRPM()
```cpp
output getRPM(float linear_x, float linear_y, float angular_z)
```
Calculates target motor RPMs from linear and angular velocities. Can be used as PID controller setpoints.

根据线速度和角速度计算各电机目标 RPM，可用于 PID 控制器的设定值。

| Parameter | Description (EN) | 描述 (CN) |
|-----------|------------------|-----------|
| `linear_x` | Linear velocity in x-axis (m/s) | x 轴线速度（m/s） |
| `linear_y` | Linear velocity in y-axis (m/s, mecanum strafing) | y 轴线速度（m/s，麦轮横移） |
| `angular_z` | Angular velocity in z-axis (rad/s) | z 轴角速度（rad/s） |

### 3. getPWM()
```cpp
output getPWM(float linear_x, float linear_y, float angular_z)
```
Same as `getRPM()` but returns PWM values.

与 `getRPM()` 类似，但返回 PWM 值。

### 4. getVelocities() - 2WD
```cpp
velocities getVelocities(int motor1, int motor2)
```
Inverse kinematics: calculates robot velocities from two motor RPMs.

逆运动学：根据两个电机的 RPM 计算机器人速度。

| Parameter | Description (EN) | 描述 (CN) |
|-----------|------------------|-----------|
| `motor1` | Left motor RPM (signed) | 左电机 RPM（带符号） |
| `motor2` | Right motor RPM (signed) | 右电机 RPM（带符号） |

### 5. getVelocities() - 4WD
```cpp
velocities getVelocities(int motor1, int motor2, int motor3, int motor4)
```
Inverse kinematics: calculates robot velocities from four motor RPMs.

逆运动学：根据四个电机的 RPM 计算机器人速度。

| Parameter | Description (EN) | 描述 (CN) |
|-----------|------------------|-----------|
| `motor1` | Front-left motor RPM | 前左电机 RPM |
| `motor2` | Front-right motor RPM | 前右电机 RPM |
| `motor3` | Rear-left motor RPM | 后左电机 RPM |
| `motor4` | Rear-right motor RPM | 后右电机 RPM |

*Motor RPM values must be signed: + for forward, - for reverse.*

*电机 RPM 需带符号：+ 表示前进，- 表示后退*

---

## Data Structures / 数据结构

### output
```cpp
struct output {
  int motor1;
  int motor2;
  int motor3;
  int motor4;
};
```

### velocities
```cpp
struct velocities {
  float linear_x;   // m/s
  float linear_y;   // m/s
  float angular_z;  // rad/s
};
```

---

## License / 许可证

This library follows the original BSD license. See the copyright notice in source files for details.

本库遵循原始 BSD 许可证。详见源文件头部的版权声明。
