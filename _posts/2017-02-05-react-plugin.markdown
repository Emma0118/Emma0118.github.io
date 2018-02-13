---
layout: post
title: React性能分析工具
date: 2017-01-05 23:13:00
---

ReactPerf是一个分析工具, 能够展示应用总体的性能概览， 并提示把钩子函数放在哪里

### 通用API

当使用react-with-addons.js在开发模式下构建的时候，这里描述的Perf对象是以React.addons.Perf的形式暴露出来的。

1. Perf.start()和Perf.stop()

开始/停止检测。React的中间操作被记录下来，用于下面的分析。花费一段微不足道的时间的操作会被忽略。

2. Perf.printInclusive(measurements)

打印花费的总时间。如果不传递任何参数，默认打印最后的所有检测记录。它会在控制台中打印一个漂亮的格式化的表格

3. Perf.printExclusive(measurements)

独占（Exclusive）”时间不包含挂载组件的时间：处理props，getInitialState，调用componentWillMount和componentDidMount，等等

4. Perf.printWasted(measurements)

“浪费”的时间被花在根本没有渲染任何东西的组件上，例如界面渲染后保持不变，没有操作DOM元素。

5. Perf.printDOM(measurements)

6. Perf.getLastMeasurements()

从最后的开始-停止会话中得到检测结果数组。该数组包含对象，每个对象看起来像这样：

```bash
{
  // The term "inclusive" and "exclusive" are explained below
  "exclusive": {},
  // '.0.0' is the React ID of the node
  "inclusive": {".0.0": 0.0670000008540228, ".0": 0.3259999939473346},
  "render": {".0": 0.036999990697950125, ".0.0": 0.010000003385357559},
  // Number of instances
  "counts": {".0": 1, ".0.0": 1},
  // DOM touches
  "writes": {},
  // Extra debugging info
  "displayNames": {
    ".0": {"current": "App", "owner": "<root>"},
    ".0.0": {"current": "Box", "owner": "App"}
  },
  "totalTime": 0.48499999684281647
}

```
