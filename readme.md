
# 0. API 功能说明

## 1. 姿态初始化
- **Topic 名称**：`initialpose`
- **消息类型**：`geometry_msgs::PoseWithCovarianceStamped`
- **功能描述**：
  - 用于在系统启动或重定位时，手动设置机器人在地图中的初始位姿。
  - 该消息包含机器人的位置（x, y, z）、朝向（四元数）以及协方差矩阵，用于表示初始位姿的不确定性。


## 2. 位姿丢失检测
- **Topic 名称**：`trajectory_localization_lost`
- **消息类型**：`std_msgs::Int8`
- **功能描述**：
  - 用于发布当前轨迹的定位状态。
  - 当消息数据为 `0` 时，表示位姿正常；为 `1` 时，表示检测到位姿丢失。
  - 频率为5HZ
- **参数配置**：
  - pose_graph.lua 中的 check_localization_lost 配置项
  - check_localization_lost.check_localization_lost_enabled
    - true 开启此性能 
    - false 关闭此功能
  - check_localization_lost.top_n_closest_submaps 计算node临近 n 个子图的匹配度
  - check_localization_lost.min_score node与临近n个子图的得分阈值， node与n个子图的匹配得分均小于min_score则认为位置丢失
- 配置示例
```lua
{
  check_localization_lost = {
    check_localization_lost_enabled = true,
    top_n_closest_submaps = 3,
    min_score = 0.4,
  }
}
```


## 3. 全局定位

