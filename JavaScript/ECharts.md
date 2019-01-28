# ECharts

`Echarts`主要由以下4部分组成

- `echarts`：全局实例，用于`echarts.init()`绑定DOM元素（容器）
- `echartsInstance`：`echarts.init()`创建的实例，其中`setOption`是非常重要的函数，有着绘制和重绘图表的功能
- `action`：一些可以操作图标的动作
- `events`：一些echarts自定义的事件，用于交互等逻辑

## 最主要的setOption


