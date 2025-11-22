# 轮式移动机器人运动学（Wheeled Mobile Robot Kinematics）

> **注意**：您正在阅读的是 ROS 2 一个较旧但仍受支持版本的文档。  
> 有关最新版本的信息，请参阅 [Kilted 版本](../../../../kilted/doc/ros2_controllers/doc/mobile_robot_kinematics.html)。

本文介绍了不同类型轮式移动机器人的运动学模型。更多参考资料请参阅：
- [*Robotics: Modelling, Planning and Control* – Siciliano 等](https://link.springer.com/book/10.1007/978-1-84628-642-1)
- [*Modern Robotics: Mechanics, Planning, and Control* – Kevin M. Lynch & Frank C. Park](http://modernrobotics.org/)

轮式移动机器人可分为两大类：

- **全向机器人**（Omnidirectional robots）：可在平面内**瞬时向任意方向移动**。
- **非完整约束机器人**（Nonholonomic robots）：**不能**在平面内瞬时向任意方向移动。

使用轮子执行器编码器对运动学模型进行前向积分，称为 **里程计定位**（odometric localization）、**被动定位**（passive localization）或 **航位推算**（dead reckoning）。本文简称为 **里程计**（odometry）。

---

## 全向轮式移动机器人（Omnidirectional Wheeled Mobile Robots）

### 使用全向轮的全向驱动机器人（Omnidirectional Drive Robots using Omni Wheels）

下文描述了使用 **3 个或更多全向轮** 的全向驱动机器人的运动学。  
坐标系遵循 [ROS REP 103](https://www.ros.org/reps/rep-0103.html) 中定义的约定。

![alt text](omni_wheel_omnidirectional_drive.svg)

#### 符号说明
- \(x_b, y_b\)：机器人本体坐标系，位于轮子与地面的接触点。
- \(x_w, y_w\)：世界坐标系。
- \(v_{b,x}\)：机器人在 x 轴方向的线速度。
- \(v_{b,y}\)：机器人在 y 轴方向的线速度。
- \(\omega_{b,z}\)：机器人绕 z 轴的角速度。
- \(R\)：机器人半径（即机器人中心到轮子的距离）。
- 红色箭头：表示第 \(i\) 个轮子旋转方向 \(\omega_i\) 为正。
- \(\gamma\)：第一个轮子相对于 \(x_b\) 轴的角度偏移。
- \(\theta\)：相邻轮子间的夹角，其中 \(n\) 为轮子数量：
  \[
  \theta = \frac{2\pi}{n}
  \]

#### 逆运动学（Inverse Kinematics）

为实现期望的本体速度（body twist），各轮所需角速度可通过以下矩阵计算：

\[
A =
\begin{bmatrix}
  \sin(\gamma) & -\cos(\gamma) & -R  \\
  \sin(\theta + \gamma) & -\cos(\theta + \gamma) & -R\\
  \sin(2\theta + \gamma) & -\cos(2\theta + \gamma) & -R\\
  \vdots & \vdots & \vdots\\
  \sin((n-1)\theta + \gamma) & -\cos((n-1)\theta + \gamma) & -R\\
\end{bmatrix}
\]

\[
\begin{bmatrix}
  \omega_1\\
  \omega_2\\
  \omega_3\\
  \vdots\\
  \omega_n
\end{bmatrix} =
\frac{1}{r}
A
\begin{bmatrix}
  v_{b,x}\\
  v_{b,y}\\
  \omega_{b,z}\\
\end{bmatrix}
\]

其中：
- \(\omega_i\)：第 \(i\) 个轮子的角速度
- \(r\)：轮子半径

任一轮子 \(i\) 的代数形式为：
\[
\omega_i = \frac{\sin((i-1)\theta + \gamma) v_{b,x} - \cos((i-1)\theta + \gamma) v_{b,y} - R \omega_{b,z}}{r}
\]

#### 正运动学（Forward Kinematics）

由轮速反推本体速度，需使用矩阵 \(A\) 的伪逆：
\[
\begin{bmatrix}
  v_{b,x}\\
  v_{b,y}\\
  \omega_{b,z}\\
\end{bmatrix} =
rA^\dagger
\begin{bmatrix}
  \omega_1\\
  \omega_2\\
  \omega_3\\
  \vdots\\
  \omega_n
\end{bmatrix}
\]

---

### 舵轮驱动机器人（Swerve Drive Robots）

下文描述了使用 **四个独立舵轮模块**（每个模块含独立转向与驱动电机）的全向机器人。  
坐标系遵循 [REP-103](https://www.ros.org/reps/rep-0103.html)。

![alt text](swerve_drive.svg)

#### 符号说明
- \(x_b, y_b\)：机器人本体坐标系，位于机器人几何中心。
- \(v_{b,x}, v_{b,y}\)：本体线速度分量。
- \(\omega_{b,z}\)：本体角速度。
- \(l\)：轴距（前后轮距离）
- \(w\)：轮距（左右轮距离）
- 红色箭头：表示第 \(i\) 个轮子的速度方向 \(v_i\)

四个模块（通常为前左、前右、后左、后右）相对于中心的位置为：
- 前左：\((l/2, w/2)\)
- 前右：\((l/2, -w/2)\)
- 后左：\((-l/2, w/2)\)
- 后右：\((-l/2, -w/2)\)

#### 逆运动学

对位于 \((l_{i,x}, l_{i,y})\) 的模块 \(i\)，其速度向量为：
\[
\begin{bmatrix}
v_{i,x} \\
v_{i,y}
\end{bmatrix}
=
\begin{bmatrix}
v_{b,x} - \omega_{b,z} l_{i,y} \\
v_{b,y} + \omega_{b,z} l_{i,x}
\end{bmatrix}
\]

轮速 \(v_i\) 与转向角 \(\phi_i\) 为：
\[
v_i = \sqrt{v_{i,x}^2 + v_{i,y}^2}, \quad
\phi_i = \arctan2(v_{i,y}, v_{i,x})
\]

#### 里程计

由轮速 \(v_i\) 和转向角 \(\phi_i\) 计算本体速度：
\[
v_{i,x} = v_i \cos(\phi_i), \quad v_{i,y} = v_i \sin(\phi_i)
\]

底盘速度为：
\[
v_{b,x} = \frac{1}{4} \sum_{i=0}^{3} v_{i,x}, \quad
v_{b,y} = \frac{1}{4} \sum_{i=0}^{3} v_{i,y}
\]
\[
\omega_{b,z} = \frac{\sum_{i=0}^{3} (v_{i,y} l_{i,x} - v_{i,x} l_{i,y})}{\sum_{i=0}^{3} (l_{i,x}^2 + l_{i,y}^2)}
\]

全局速度用于更新位姿 \((x, y, \theta)\)：
\[
v_{x,\text{global}} = v_{b,x} \cos(\theta) - v_{b,y} \sin(\theta)
\]
\[
v_{y,\text{global}} = v_{b,x} \sin(\theta) + v_{b,y} \cos(\theta)
\]

---

## 非完整约束轮式移动机器人（Nonholonomic Wheeled Mobile Robots）

### 单轮车模型（Unicycle Model）

遵循 [ROS 坐标系约定](https://www.ros.org/reps/rep-0103.html#id19)（右手定则）：

![alt text](unicycle.svg)
#### 符号说明
- \(x_b, y_b\)：本体坐标系（位于轮地接触点）
- \(x, y\)：世界坐标系中的机器人位置
- \(\theta\)：机器人航向角（本体 \(x_b\) 轴与世界 \(x_w\) 轴夹角）

期望的本体速度为：
\[
\vec{\nu}_b = 
\begin{bmatrix}
\vec{\omega}_{b} \\
\vec{v}_{b}
\end{bmatrix}
\]
其中：
- \(v_{b,x}\)：沿 \(x_b\) 轴的线速度
- \(\omega_{b,z}\)：绕 \(z\) 轴的角速度

#### 正运动学
\[
\dot{x} = v_{b,x} \cos(\theta), \quad
\dot{y} = v_{b,x} \sin(\theta), \quad
\dot{\theta} = \omega_{b,z}
\]

---

### 差速驱动机器人（Differential Drive Robot）

> 引自 Siciliano 等：  
> “严格意义上的单轮车（即单轮车辆）在静态条件下存在严重平衡问题。然而，存在一些在运动学上等效于单轮车但机械更稳定的车辆。”

差速驱动机器人有两个**独立驱动**的轮子。

![alt text](diff_drive.svg)

- \(w\)：轮距（两轮间距）

#### 正运动学
\[
v_{b,x} = \frac{v_{\text{right}} + v_{\text{left}}}{2}, \quad
\omega_{b,z} = \frac{v_{\text{right}} - v_{\text{left}}}{w}
\]

#### 逆运动学
\[
v_{\text{left}} = v_{b,x} - \omega_{b,z} \frac{w}{2}, \quad
v_{\text{right}} = v_{b,x} + \omega_{b,z} \frac{w}{2}
\]

#### 里程计
直接使用正运动学方程，由编码器读数计算位姿。

---

### 类车模型（自行车模型）（Car-Like / Bicycle Model）

前轮可转向的两轮机器人（又称自行车模型）：

![alt text](car_like_robot.svg)

#### 符号说明
- \(\phi\)：前轮转向角（绕 \(z\) 轴为正）
- \(v_{\text{rear}}, v_{\text{front}}\)：后轮与前轮速度
- \(l\)：轴距

假设**无滑移**，则两轮速度满足：
\[
v_{\text{rear}} = v_{\text{front}} \cos(\phi)
\]

#### 正运动学
\[
\dot{x} = v_{b,x} \cos(\theta), \quad
\dot{y} = v_{b,x} \sin(\theta), \quad
\dot{\theta} = \frac{v_{b,x}}{l} \tan(\phi)
\]

#### 逆运动学
- 转向角：\(\phi = \arctan\left(\frac{l \omega_{b,z}}{v_{b,x}} \right)\)
- 后轮驱动：\(v_{\text{rear}} = v_{b,x}\)
- 前轮驱动：\(v_{\text{front}} = \frac{v_{b,x}}{\cos(\phi)}\)

#### 里程计
- **后轮编码器**：
  \[
  \dot{x} = v_{\text{rear}} \cos(\theta),\ 
  \dot{y} = v_{\text{rear}} \sin(\theta),\ 
  \dot{\theta} = \frac{v_{\text{rear}}}{l} \tan(\phi)
  \]
- **前轮编码器**：
  \[
  \dot{x} = v_{\text{front}} \cos(\theta) \cos(\phi),\ 
  \dot{y} = v_{\text{front}} \sin(\theta) \cos(\phi),\ 
  \dot{\theta} = \frac{v_{\text{front}}}{l} \sin(\phi)
  \]

---

### 双驱动后轴（Double-Traction Axle）

三轮类车机器人，后轴有两个独立驱动轮：

![alt text](double_traction.svg)


- \(w_r\)：后轴轮距

#### 逆运动学
转弯半径：\(R_b = \frac{l}{\tan(\phi)}\)

为避免滑移，后轮速度需满足：
\[
v_{\text{rear,left}} = v_{b,x}\frac{R_b - w_r/2}{R_b}, \quad
v_{\text{rear,right}} = v_{b,x}\frac{R_b + w_r/2}{R_b}
\]

#### 里程计
由两个编码器测得的 \(v_{b,x}\) 是超定的。取平均以提高鲁棒性：
\[
v_{b,x} = \frac{1}{2} \left(v_{\text{rear,left}} \frac{R_b}{R_b - w_r/2} + v_{\text{rear,right}} \frac{R_b}{R_b + w_r/2}\right)
\]

---

### 阿克曼转向（Ackermann Steering）

四轮机器人，前轴有两个**独立转向轮**：

![alt text](ackermann_steering.svg)

- \(w_f\)：前轴轮距（两主销间距离）

> **注意**：阿克曼转向也可通过**机械连杆**实现（仅一个转向输入），此时运动学与类车模型相同。

#### 逆运动学
转弯半径：\(R_b = \frac{l}{\tan(\phi)}\)

为避免滑移，前轮转向角需满足：
\[
\phi_{\text{left}} = \arctan\left(\frac{l}{R_b - w_f/2}\right), \quad
\phi_{\text{right}} = \arctan\left(\frac{l}{R_b + w_f/2}\right)
\]

#### 里程计
由两个转向角测得的 \(\phi\) 是超定的，取平均：
\[
\phi = \frac{1}{2} \left( 
\arctan\left(\frac{l\tan(\phi_{\text{left}})}{l + \frac{w_f}{2} \tan(\phi_{\text{left}})}\right) +
\arctan\left(\frac{l\tan(\phi_{\text{right}})}{l - \frac{w_f}{2} \tan(\phi_{\text{right}})}\right)
\right)
\]

---

### 带驱动的阿克曼转向（Ackermann Steering with Traction）

前轮既可转向又可独立驱动的四轮类车机器人：

![alt text](ackermann_steering_traction.svg)

- \(d_{kp}\)：主销到前轮接地接触点的距离

#### 逆运动学
为避免滑移，前轮速度需满足：
\[
\frac{v_{\text{front,left}}}{R_{\text{left}}} = \frac{v_{\text{front,right}}}{R_{\text{right}}} = \frac{v_{b,x}}{R_b}
\]
其中：
\[
R_b = \frac{l}{\tan(\phi)},\quad
R_{\text{left}} = \frac{l - d_{kp}\sin(\phi_{\text{left}})}{\sin(\phi_{\text{left}})},\quad
R_{\text{right}} = \frac{l + d_{kp}\sin(\phi_{\text{right}})}{\sin(\phi_{\text{right}})}
\]

解得：
\[
v_{\text{front,left}} = \frac{v_{b,x}(l - d_{kp}\sin(\phi_{\text{left}}))}{R_b \sin(\phi_{\text{left}})},\quad
v_{\text{front,right}} = \frac{v_{b,x}(l + d_{kp}\sin(\phi_{\text{right}}))}{R_b \sin(\phi_{\text{right}})}
\]

#### 里程计
同样取平均提高鲁棒性：
\[
v_{b,x} = \frac{1}{2} \left( 
v_{\text{front,left}} \frac{R_b \sin(\phi_{\text{left}})}{l - d_{kp}\sin(\phi_{\text{left}})} +
v_{\text{front,right}} \frac{R_b \sin(\phi_{\text{right}})}{l + d_{kp}\sin(\phi_{\text{right}})}
\right)
\]