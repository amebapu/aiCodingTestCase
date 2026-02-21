# aiCodingTestCase

# Case 1 Prompt：多电梯智能调度模拟器

> 以下为发给各模型的完整 Prompt，直接复制粘贴使用。

---

请用 **一个 HTML 文件** 实现一个多电梯智能调度模拟系统。要求如下：

## 场景设定

- 一栋 40 层的写字楼（1-40 楼），配备 **6 部电梯**
- 每部电梯最大载客 **15 人**
- 电梯移动速度：每层 1.5 秒；开关门耗时 3 秒；乘客进出每人 1 秒
- 在大厅（1楼）和各楼层均有呼叫按钮（上行/下行）

## 核心功能

### 1. 调度算法
设计一个高效的多电梯调度算法，**最终优化目标是最小化所有乘客的平均等待时间和平均乘梯时间，同时确保 6 部电梯都能参与运转、负载均衡**。需要考虑：
- 距离优先：优先分配最近的空闲电梯
- 顺路搭载：同方向运行中的电梯可以顺路接人
- 负载均衡：避免所有乘客挤在同一部电梯，6 部电梯应合理分工
- 满载跳过：已满载的电梯不响应新的楼层请求

### 2. 模拟引擎
- 时间步进式模拟：以 0.5 秒为一个时间步
- 支持加载预设的乘客到达场景（JSON 格式）
- 每个乘客有：到达时间（秒）、出发楼层、目标楼层
- **必须确保所有乘客最终都能被送达目标楼层**，不允许丢失乘客

### 3. 可视化界面
- 左侧：6 部电梯的实时动画（显示当前楼层、运行方向箭头、载客人数）
- 右侧：实时统计面板，显示：
  - 已运送乘客数 / 等待中乘客数
  - 当前平均等待时间
  - 最长等待时间
  - 每部电梯的状态（空闲/上行/下行/开门）
- 底部：时间轴进度条，显示模拟时间
- 楼层标签从下到上排列（1楼在底部，40楼在顶部）

### 4. 控制面板
- 开始/暂停/重置按钮
- 模拟速度切换：1x / 5x / 10x / 20x
- 可以手动添加乘客（选择出发楼层和目标楼层）

## 技术要求

### 必须暴露的全局接口

你的代码**必须**在 window 对象上暴露以下接口，这是自动化测试的硬性要求：

```javascript
window.ElevatorSystem = {
  /**
   * 加载测试场景
   * @param {Array} scenario - 乘客列表
   * 格式: [{ time: 0, from: 1, to: 15 }, { time: 2, from: 1, to: 28 }, ...]
   * time: 到达时间（秒），from: 出发楼层，to: 目标楼层
   */
  loadScenario: function(scenario) { },

  /**
   * 运行完整模拟（快进到所有乘客送达）
   * @returns {Promise} 完成时 resolve
   */
  runSimulation: function() { },

  /**
   * 获取模拟结果指标
   * @returns {Object} {
   *   avgWaitTime: number,    // 平均等待时间（秒）
   *   maxWaitTime: number,    // 最长等待时间（秒）
   *   avgTravelTime: number,  // 平均乘梯时间（秒）
   *   totalDelivered: number, // 已送达乘客数
   *   avgTotalTime: number    // 平均总耗时（等待+乘梯）
   * }
   */
  getMetrics: function() { },

  /**
   * 获取完整事件日志（用于外部独立验证）
   * @returns {Array} [
   *   { time: 0, type: 'passenger_arrived', detail: { id: 1, from: 1, to: 15 } },
   *   { time: 5, type: 'elevator_moved', detail: { elevatorId: 1, from: 3, to: 4 } },
   *   { time: 8, type: 'passenger_entered', detail: { passengerId: 1, elevatorId: 1, floor: 1 } },
   *   { time: 20, type: 'passenger_exited', detail: { passengerId: 1, elevatorId: 1, floor: 15 } },
   *   ...
   * ]
   */
  getEventLog: function() { }
};
```

### 设计要求
- 使用 Canvas 或 DOM 进行电梯动画渲染
- 科技感配色
- 动画流畅，不卡顿
- 统计数据实时更新

## 内置演示场景

代码中需要内置以下**固定的**"早高峰"演示场景数据（直接硬编码，不要用随机数生成）。页面加载后自动加载此场景，用户点击"开始"即可运行：

```javascript
const DEFAULT_SCENARIO = [
  {time:0,from:1,to:15},{time:1,from:1,to:28},{time:2,from:1,to:7},{time:3,from:1,to:35},
  {time:4,from:1,to:12},{time:5,from:1,to:22},{time:6,from:1,to:38},{time:7,from:1,to:3},
  {time:8,from:1,to:19},{time:9,from:1,to:31},{time:10,from:1,to:8},{time:11,from:1,to:25},
  {time:12,from:1,to:14},{time:14,from:1,to:40},{time:15,from:1,to:6},{time:16,from:1,to:33},
  {time:18,from:1,to:11},{time:19,from:1,to:27},{time:20,from:1,to:17},{time:22,from:1,to:36},
  {time:23,from:1,to:4},{time:24,from:1,to:21},{time:26,from:1,to:9},{time:27,from:1,to:30},
  {time:28,from:1,to:16},{time:30,from:1,to:39},{time:31,from:1,to:13},{time:33,from:1,to:24},
  {time:34,from:1,to:5},{time:35,from:1,to:37},{time:37,from:1,to:10},{time:38,from:1,to:29},
  {time:40,from:1,to:18},{time:42,from:1,to:34},{time:43,from:1,to:2},{time:45,from:1,to:23},
  {time:46,from:1,to:20},{time:48,from:1,to:32},{time:50,from:1,to:26},{time:52,from:1,to:8},
  {time:53,from:1,to:15},{time:55,from:1,to:40},{time:56,from:1,to:11},{time:58,from:1,to:37},
  {time:60,from:1,to:6},{time:62,from:1,to:22},{time:63,from:1,to:19},{time:65,from:1,to:33},
  {time:67,from:1,to:14},{time:70,from:1,to:28},{time:72,from:1,to:3},{time:74,from:1,to:35},
  {time:75,from:1,to:17},{time:78,from:1,to:30},{time:80,from:1,to:9},{time:82,from:1,to:25},
  {time:85,from:1,to:12},{time:87,from:1,to:38},{time:90,from:1,to:7},{time:93,from:1,to:21},
  {time:95,from:1,to:16},{time:98,from:1,to:36},{time:100,from:1,to:10},{time:103,from:1,to:27},
  {time:105,from:1,to:4},{time:108,from:1,to:31},{time:110,from:1,to:13},{time:113,from:1,to:39},
  {time:115,from:1,to:24},{time:118,from:1,to:5},{time:120,from:1,to:34},{time:123,from:1,to:18},
  {time:125,from:1,to:29},{time:128,from:1,to:2},{time:130,from:1,to:20},{time:133,from:1,to:32},
  {time:135,from:1,to:23},{time:138,from:1,to:8},{time:140,from:1,to:40},{time:143,from:1,to:11},
  {time:145,from:1,to:26},{time:148,from:1,to:15},{time:150,from:1,to:37},{time:153,from:1,to:6},
  {time:155,from:1,to:19},{time:158,from:1,to:33},{time:160,from:1,to:14},{time:163,from:1,to:28},
  {time:165,from:1,to:7},{time:168,from:1,to:35},{time:170,from:1,to:22},{time:173,from:1,to:9},
  {time:175,from:1,to:30},{time:178,from:1,to:16},{time:180,from:1,to:38},{time:183,from:1,to:3},
  {time:185,from:1,to:25},{time:188,from:1,to:12},{time:190,from:1,to:36},{time:193,from:1,to:17}
];
```

以上为 100 名乘客在 0-193 秒内从 1 楼出发去往各楼层的早高峰场景。

## 交付

输出一个完整的、可直接在浏览器中打开运行的 HTML 文件。所有 CSS 和 JavaScript 都内联在该文件中。不要使用任何外部依赖或 CDN。
