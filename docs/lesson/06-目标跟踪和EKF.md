# 第 6 课：目标跟踪和 EKF

## 本课目标

- 理解 `armor_processor` 如何把一帧帧装甲板检测结果融合成稳定目标。
- 理解 tracker 状态机。
- 理解 EKF 9 维状态的含义。

## 前置知识

- 已完成第 0 到第 5 课。
- 知道 `Armor.pose` 和 `Target.position` 的区别。

## 阅读文件

- `auto_aim_hero_old/src/auto_aim/armor_processor/src/processor_node.cpp`
- `auto_aim_hero_old/src/auto_aim/armor_processor/include/armor_processor/processor_node.hpp`
- `auto_aim_hero_old/src/auto_aim/armor_processor/src/tracker.cpp`
- `auto_aim_hero_old/src/auto_aim/armor_processor/include/armor_processor/tracker.hpp`
- `auto_aim_hero_old/src/auto_aim/armor_processor/src/extended_kalman_filter.cpp`
- `auto_aim_hero_old/src/auto_aim/armor_processor/include/armor_processor/extended_kalman_filter.hpp`

## 课堂内容

### 1. processor 输入输出

订阅：

```text
/detector/armors
```

发布：

```text
/processor/target
/tracker/measurement
/processor/marker
```

### 2. 坐标变换

processor 会把装甲板位姿从图像/相机相关坐标系变换到 `target_frame`，默认通常是 `world`。

如果 TF 不可用，tracker 无法稳定工作。

### 3. tracker 状态机

状态包括：

- `LOST`
- `DETECTING`
- `TRACKING`
- `TEMP_LOST`

基本逻辑：

```text
LOST
  -> 有装甲板，初始化
DETECTING
  -> 连续匹配成功，进入 TRACKING
TRACKING
  -> 匹配失败，进入 TEMP_LOST
TEMP_LOST
  -> 恢复匹配，回到 TRACKING
  -> 丢失太久，回到 LOST
```

### 4. EKF 状态

EKF 状态是 9 维：

```text
xc, v_xc, yc, v_yc, zc, v_zc, yaw, v_yaw, r
```

含义：

- `xc, yc, zc`：目标中心位置。
- `v_xc, v_yc, v_zc`：目标中心速度。
- `yaw`：目标旋转角。
- `v_yaw`：目标旋转角速度。
- `r`：目标中心到当前装甲板的半径。

### 5. Target 消息

`Target` 是给 `solve_angle` 使用的高层目标状态。它包含目标中心、速度、旋转状态、半径和是否正在跟踪。

## 实践任务

1. 画出 tracker 状态机。
2. 写出 EKF 9 维状态表：

```text
索引 | 名称 | 含义 | 对应 Target 字段
```

3. 找出 `node_params.yaml` 中 processor 相关参数。
4. 记录这些参数的意义：

- `max_armor_distance`
- `tracker.max_match_distance`
- `tracker.max_match_yaw_diff`
- `tracker.tracking_thres`
- `tracker.lost_time_thres`
- `ekf.sigma2_q_xyz`
- `ekf.sigma2_q_yaw`
- `ekf.r_xyz_factor`
- `ekf.r_yaw`

5. 设计一个观察 tracker 状态变化的 debug 方案。

## 检查问题

- `Armor.pose` 如何变成 `Target.position`？
- tracker 为什么需要 `DETECTING` 状态，而不是一看到装甲板就 `TRACKING`？
- `max_match_distance` 太大或太小分别会带来什么问题？
- `v_yaw` 对后续弹道预测有什么意义？
- `Target.tracking == false` 时，`solve_angle` 应该如何处理？

## 课程完成条件

完成本课需要同时满足：

- 能画出 tracker 四个状态及转换条件。
- 能解释 EKF 9 维状态的每一维。
- 能说明 `Target` 中位置、速度、yaw、v_yaw、radius 的来源。
- 能列出 processor/tracker 至少 8 个关键参数及其作用。
- 能写出“有识别但没有稳定目标”的排查顺序，至少包含 TF、匹配阈值、tracker 状态和 `/processor/target`。
