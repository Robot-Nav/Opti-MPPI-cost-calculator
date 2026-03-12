# MPPI 批评函数代价公式及计算工具

## 在线计算器

📱 **[点击打开 MPPI 批评函数代价计算器](critic_cost_calculator.html)**

实时输入参数，即时计算各批评函数的代价值！

***

## 使用说明

将实际参数代入公式，计算各批评函数的代价值。所有代价最终累加到总代价中。

***

## 1. ObstaclesCritic (障碍物代价)

### 参数

```yaml
obstacle_repulsion_weight: 1.0      # w_rep
obstacle_cost_scaling: 6.0          # k
obstacle_collision_margin: 0.4      # d_margin
obstacle_inflation_radius: 0.8      # d_inflation
robot_radius: 0.30                  # r_robot
```

### 公式

**排斥代价** (当 $d\_{margin} < d\_{min} < d\_{inflation}$ 时):

$$d = d\_{min} - d\_{margin}$$

$$cost = w\_{rep} \cdot e^{-k \cdot d}$$

**碰撞代价** (当 $d\_{min} < d\_{margin}$ 时):

$$cost = cost\_{collision} = 10000$$

### 示例计算

- $d\_{min} = 0.5m$ (到障碍物距离)
- $d = 0.5 - 0.4 = 0.1$
- $cost = 1.0 \cdot e^{-6.0 \cdot 0.1} = 1.0 \cdot 0.5488 = \mathbf{0.55}$

***

## 2. PathFollowCritic (路径跟随代价)

### 参数

```yaml
path_follow_weight: 5.0             # w_pf
path_follow_offset: 6               # offset
path_follow_threshold: 0.5          # threshold
```

### 公式

$$dist = \sqrt{(x\_{traj\_end} - x\_{path\_ref})^2 + (y\_{traj\_end} - y\_{path\_ref})^2}$$

$$cost = w\_{pf} \cdot dist$$

### 示例计算

- 轨迹终点偏离路径参考点 0.3m
- $cost = 5.0 \cdot 0.3 = \mathbf{1.5}$

***

## 3. PathAlignCritic (路径对齐代价)

### 参数

```yaml
path_align_weight: 10.0             # w_pa
path_align_offset: 8                # offset
path_align_threshold: 0.4           # threshold
path_align_traj_step: 2             # step
```

### 公式

对于轨迹上的每个采样点 $j$:

$$point\_dist\_j = \sqrt{(x\_{traj}\[j] - x\_{path}\[nearest])^2 + (y\_{traj}\[j] - y\_{path}\[nearest])^2}$$

如果启用路径朝向考虑:

$$point\_dist\_j = point\_dist\_j \cdot (1 + 0.5 \cdot |\Delta\theta|)$$

其中 $|\Delta\theta| = |shortestAngularDistance(\theta\_{traj}\[j], \theta\_{path}\[nearest])|$

$$cost = w\_{pa} \cdot \frac{1}{N} \sum\_{j=1}^{N} point\_dist\_j$$

### 示例计算

- 采样3个点，到路径距离分别为: 0.1m, 0.2m, 0.15m
- $avg\_dist = (0.1 + 0.2 + 0.15) / 3 = 0.15m$
- $cost = 10.0 \cdot 0.15 = \mathbf{1.5}$

***

## 4. PathAngleCritic (路径角度代价)

### 参数

```yaml
path_angle_weight: 5.0              # w_pang
path_angle_offset: 4                # offset
path_angle_threshold: 0.3           # threshold
path_angle_max: 0.15                # max_angle (rad)
path_angle_mode: 0                  # mode (0/1/2)
```

### 公式 (Mode 0: 偏好前进)

$$\theta\_{target} = atan2(y\_{target} - y\_{traj}, x\_{target} - x\_{traj})$$

$$\Delta\theta = |shortestAngularDistance(\theta\_{traj}, \theta\_{target})|$$

$$cost = \begin{cases} w\_{pang} \cdot \Delta\theta & \text{if } \Delta\theta > \theta\_{max} \ 0 & \text{otherwise} \end{cases}$$

### 示例计算

- 朝向偏差 0.3 rad (> 0.15)
- $cost = 5.0 \cdot 0.3 = \mathbf{1.5}$

***

## 5. GoalCritic (目标点代价)

### 参数

```yaml
goal_weight: 5.0                    # w_g
goal_threshold: 0.8                 # threshold
```

### 公式 (仅在 $d\_{goal} < threshold$ 时激活)

$$dist = \sqrt{(x\_{traj\_end} - x\_{goal})^2 + (y\_{traj\_end} - y\_{goal})^2}$$

$$cost = w\_g \cdot dist^2$$

### 示例计算

- 距离目标 0.5m
- $cost = 5.0 \cdot (0.5)^2 = 5.0 \cdot 0.25 = \mathbf{1.25}$

***

## 6. GoalAngleCritic (目标角度代价)

### 参数

```yaml
goal_angle_weight: 3.0              # w_ga
goal_angle_threshold: 0.6           # threshold
```

### 公式 (仅在 $d\_{goal} < threshold$ 时激活)

$$\Delta\theta = |shortestAngularDistance(\theta\_{traj}, \theta\_{goal})|$$

$$cost = w\_{ga} \cdot \Delta\theta^2$$

### 示例计算

- 朝向偏差 0.2 rad
- $cost = 3.0 \cdot (0.2)^2 = 3.0 \cdot 0.04 = \mathbf{0.12}$

***

## 7. PreferForwardCritic (偏好前进代价)

### 参数

```yaml
prefer_forward_weight: 3.0          # w_pfwd
prefer_forward_threshold: 0.5       # threshold
```

### 公式 (仅在 $d\_{goal} > threshold$ 时激活)

$$backward\_cost = \sum\_{j=1}^{N} |v\_x\[j]| \quad \text{for all } v\_x\[j] < 0$$

$$cost = w\_{pfwd} \cdot backward\_cost$$

### 示例计算

- 轨迹中有2个负速度点: -0.1, -0.05
- $backward\_cost = 0.1 + 0.05 = 0.15$
- $cost = 3.0 \cdot 0.15 = \mathbf{0.45}$

***

## 8. ConstraintCritic (约束代价)

### 参数

```yaml
constraint_weight: 3.0              # w_c
vx_max: 1.2
vx_min: -0.25
wz_max: 2.0
motion_model_type: 1                # 0=Diff, 1=Omni, 2=Ackermann
```

### 公式 (DiffDrive模式)

$$vx\_{violation} = max(0, v\_x - vx\_{max}) + max(0, vx\_{min} - v\_x)$$

$$wz\_{violation} = max(0, |w\_z| - wz\_{max})$$

$$cost\_{per\_step} = (vx\_{violation} + wz\_{violation}) \cdot dt$$

$$cost = w\_c \cdot \frac{1}{T} \sum\_{t=1}^{T} cost\_{per\_step}\[t]$$

### 公式 (Omni模式)

$$v\_{total} = sign(v\_x) \cdot \sqrt{v\_x^2 + v\_y^2}$$

$$vel\_{violation} = max(0, v\_{total} - vx\_{max}) + max(0, vx\_{min} - v\_{total})$$

$$wz\_{violation} = max(0, |w\_z| - wz\_{max})$$

$$cost\_{per\_step} = (vel\_{violation} + wz\_{violation}) \cdot dt$$

### 示例计算

- $v\_x = 1.5 (> 1.2)$, $w\_z = 2.5 (> 2.0)$
- $vx\_{violation} = 1.5 - 1.2 = 0.3$
- $wz\_{violation} = 2.5 - 2.0 = 0.5$
- $cost\_{per\_step} = (0.3 + 0.5) \cdot 0.05 = 0.04$
- $cost = 3.0 \cdot 0.04 = \mathbf{0.12}$

***

## 9. TwirlingCritic (旋转惩罚代价)

### 参数

```yaml
twirling_weight: 0.5                # w_twirl
vx_max: 1.2                         # 用于归一化
twirling_threshold: 0.5             # threshold
```

### 公式 (仅在 $d\_{goal} > threshold$ 时激活)

$$forward\_ratio = clamp\left(\frac{v\_x}{vx\_{max}}, 0, 1\right)$$

$$spin\_penalty = (1 - forward\_ratio) \cdot |w\_z|$$

$$cost = w\_{twirl} \cdot \frac{1}{T} \sum\_{t=1}^{T} spin\_penalty\[t]$$

### 示例计算

- $v\_x = 0.1$, $w\_z = 1.0$, $vx\_{max} = 1.2$
- $forward\_ratio = 0.1 / 1.2 = 0.083$
- $spin\_penalty = (1 - 0.083) \cdot 1.0 = 0.917$
- $cost = 0.5 \cdot 0.917 = \mathbf{0.46}$

***

## 10. VelocityDeadbandCritic (速度死区代价)

### 参数

```yaml
velocity_deadband_weight: 0.5       # w_vd
velocity_deadband_vx: 0.05          # db_vx
velocity_deadband_vy: 0.05          # db_vy
velocity_deadband_wz: 0.1           # db_wz
```

### 公式

$$vx\_{penalty} = max(0, db\_{vx} - |v\_x|)$$

$$vy\_{penalty} = max(0, db\_{vy} - |v\_y|)$$

$$wz\_{penalty} = max(0, db\_{wz} - |w\_z|)$$

$$cost\_{per\_step} = (vx\_{penalty} + vy\_{penalty} + wz\_{penalty}) \cdot dt$$

$$cost = w\_{vd} \cdot \frac{1}{T} \sum\_{t=1}^{T} cost\_{per\_step}\[t]$$

### 示例计算

- $v\_x = 0.02 (< 0.05)$, $v\_y = 0$, $w\_z = 0.05 (< 0.1)$
- $vx\_{penalty} = 0.05 - 0.02 = 0.03$
- $wz\_{penalty} = 0.1 - 0.05 = 0.05$
- $cost\_{per\_step} = (0.03 + 0 + 0.05) \cdot 0.05 = 0.004$
- $cost = 0.5 \cdot 0.004 = \mathbf{0.002}$

***

## 快速计算表

| 批评函数                   | 主要参数      | 典型代价值范围 | 调节建议           |
| ---------------------- | --------- | ------- | -------------- |
| ObstaclesCritic        | w\_rep, k | 0-10    | 避障不积极时增大w\_rep |
| PathFollowCritic       | w\_pf     | 0-5     | 轨迹终点不准时增大      |
| PathAlignCritic        | w\_pa     | 0-5     | 轨迹不贴合路径时增大     |
| PathAngleCritic        | w\_pang   | 0-2     | 朝向偏差大时增大       |
| GoalCritic             | w\_g      | 0-5     | 接近目标时激活        |
| GoalAngleCritic        | w\_ga     | 0-2     | 目标朝向不准时增大      |
| PreferForwardCritic    | w\_pfwd   | 0-2     | 避免倒车时增大        |
| ConstraintCritic       | w\_c      | 0-1     | 违反约束时产生        |
| TwirlingCritic         | w\_twirl  | 0-5     | 防止原地旋转         |
| VelocityDeadbandCritic | w\_vd     | 0-0.1   | 避免低速不稳定        |

***

## 总代价计算

$$Total\_Cost = \sum\_{i=1}^{10} Cost\_i$$

即:

$$Total\_Cost = Cost\_{obstacles} + Cost\_{path\_follow} + Cost\_{path\_align} + Cost\_{path\_angle} + Cost\_{goal} + Cost\_{goal\_angle} + Cost\_{prefer\_forward} + Cost\_{constraint} + Cost\_{twirling} + Cost\_{velocity\_deadband}$$

MPPI选择总代价最小的轨迹作为最优轨迹。

***

## 辅助函数

### 角度规范化

$$normalizeAngle(\theta) = \theta - 2\pi \cdot round\left(\frac{\theta}{2\pi}\right)$$

### 最短角度距离

$$shortestAngularDistance(from, to) = normalizeAngle(to - from)$$

结果范围: $\[-\pi, \pi]$

### 限幅函数

$$clamp(val, min, max) = \begin{cases} min & \text{if } val < min \ max & \text{if } val > max \ val & \text{otherwise} \end{cases}$$
