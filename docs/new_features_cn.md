# New Features in the Ad Data Closed Loop
Welcome to the documentation for the new features introduced in the Ad Data Closed Loop system. This document outlines the latest enhancements and functionalities that have been added to improve user experience and system performance.

## 导航规划器
输入：静态地图、实时costmap、采集密度热图，输出最优路径规划
输出：覆盖关键稀疏区域的路径（waypoints | full trajectory）
策略：在costmap基础上对稀疏区域降低代价（鼓励访问），对高密度区域提高代价（鼓励避让）

## 数据闭环
提取bag中pose、相机视点、标签覆盖等计算采集密度热图和统计特征，输出新权重文件（weights_file）并通知planner重载
planner_weights_file.yaml:
```
sparse_threshold: 0.2 //低于该阈值的区域被认为是稀疏区域
exploration_bonus: 0.5
redundancy_penalty: 0.4
grid_resolution: 1.0
```

## 策略
1. **Reward设计**：（ 鼓励采集稀疏区域数据，惩罚无效移动与碰撞）
- 到达未采样/稀疏cell：大量正奖励 +10
- 触发成功（触发器产生有用数据）：额外奖励（+0.5）
- 每步小惩罚（-0.01）鼓励短路径
- 碰撞/进入障碍：大惩罚（-1.0）
- 轨迹多样性/不重复采样：鼓励访问新cells

2. **量化指标**:
AB测试记录指标：
- 采集覆盖率（grid cells visited / total cells）
- 稀疏样本覆盖率（稀疏cells visited / total sparse cells）
- Trigger成功率（触发次数/有价值数据数）
- 单次任务效率（单位时间内有效样本数）
- 路线重复率（轨迹相似性度量）
- 安全指标（碰撞次数/interupted missions）

## 接口
| Topic           | Type                        | 说明                        | 
| --------------- | --------------------------- | --------------------------- |
| /planner/request  | std_msgs/String       | A*规划请求"sx, sy, gx, gy"
| /planned_path | nav_msgs/Path | A*输出路径
| /trigger | ad_msgs/TriggerContext | 触发采集，rosbag自动记录
| /planner/next_waypoint | geometry_msgs/PointStamped | RL输出连续waypoint
| /planner/re_plan | std_msgs/Bool | 参数/模型reload

## 实现
```python
"""
奖励函数
"""
def compute_reward(prev_state, new_state, info):
    rew = 0.0
    if info.get('visited_new_sparse', False):
        rew += 1.0
    if info.get('trigger_success', False):
        rew += 0.5
    if info.get('collision', False):
        rew -= 1.0
    rew -= 0.01  # step cost
    # optionally shaped reward: distance to nearest sparse cell
    return rew

```
```cpp
/**
 *@brief 根据历史采集数据的分布密度动态调整成本地图
 *       可视化输出Rviz中显示稀疏采样区（红色）、高密度区（蓝色）
 */
void Planner::optRoute(const MapData& map, const DataStats& stats) {
    CostMap costmap = map.toCostMap();
    for (auto& cell : costmap.cells) {
        if (stats.dataDensity(cell.position) < threshold_sparse)
            cell.cost -= exploration_bonus;
        else 
            cell.cost += redundancy_penalty;
    }
    current_route = AStar::compute(costmap, start, goal);
}

```

```python
"""
数据采集完成生成空间分布统计
根据“采样稀疏度”生成新的costmap
结果写回planner_wights_file.yaml
下次任务启动时自动加载重规划
"""
def update_planner_weights(data_map):
    heatmap = compute_density_map(data_map)
    sparse_zones = detect_sparse_regions(heatmap)
    new_weights = adjust_cost_weights(sparse_zones)
    save_to_planner_config(new_wights)
```





