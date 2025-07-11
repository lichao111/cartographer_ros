
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

- **Topic 名称**：`global_relocalization`
- **消息类型**：`std_msgs::Empty`
- **功能描述**：
  - 用于在机器人定位丢失或需要重新定位时，主动触发全局重定位流程。
  - 客户端只需向该 topic 发布一条空消息即可触发 Cartographer 的全局定位功能。
  - Cartographer 会自动搜索所有已知子图，计算当前激光点云与地图的最佳匹配位置，并以此为初始位姿开启新的轨迹。
  - 该功能适用于机器人在地图中“迷路”或需要恢复定位的场景。

- **状态反馈**：
  - **Topic 名称**：`global_relocalization_status`
  - **消息类型**：`std_msgs::Int8`
    - `0`：全局定位成功
    - `-1`：全局定位失败
  - Cartographer 在全局定位完成后会通过该 topic 发布定位结果，便于客户端或上层系统做进一步处理。

- **典型用法**：

  在代码中发布：
  ```python
    rospy.init_node('global_relocalization_sender')
    pub = rospy.Publisher('/global_relocalization', Empty, queue_size=1)
    rospy.sleep(1)  # 等待话题连接
    pub.publish(Empty())
    rospy.loginfo("Sent std_msgs/Empty to /global_relocalization")
  ```

- **注意事项**：
  - `rostopic pub /global_relocalization std_msgs/Empty -1` 命令不可用，会导致消息latched, 代码中重复消费
  - 触发后会自动关闭所有活动轨迹，并以新定位结果开启新轨迹。
  - 改接口的耗时除与算力相关外，还取决于地图大小