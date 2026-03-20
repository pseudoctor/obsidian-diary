# Obsidian 日志模板

一套面向 Obsidian 的五级日志模板，覆盖日报、周报、月报、季报和年报，并配套 DataviewJS 自动汇总脚本说明。

适合想把「每日执行记录」和「长期复盘」串起来的人。模板默认偏向个人管理、习惯追踪、阅读记录、项目复盘和投资复盘。

## 包含内容

- [01a1 Daily Dairy.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/01a1%20Daily%20Dairy.md)：日报模板，包含时间块、习惯追踪、任务管理、投资复盘、项目复盘等
- [01a2 Weekly Dairy.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/01a2%20Weekly%20Dairy.md)：周报模板，支持从日报自动聚合
- [01a3 Monthly Dairy.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/01a3%20Monthly%20Dairy.md)：月报模板，支持月度趋势汇总
- [01a4 Quarterly Dairy.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/01a4%20Quarterly%20Dairy.md)：季报模板，支持季度对比
- [01a5 Yearly Dairy.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/01a5%20Yearly%20Dairy.md)：年报模板，支持年度统计与季度趋势
- [00_Template_User_Guide.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/00_Template_User_Guide.md)：详细使用说明
- [00_AutoAggregation_Scripts.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/00_AutoAggregation_Scripts.md)：自动汇总脚本说明

## 依赖要求

建议至少安装以下 Obsidian 插件：

- `Templater`
- `Dataview`

可选但推荐：

- `Hover Editor`

模板中的 `<% ... %>` 语法依赖 `Templater`。  
周/月/季/年报中的自动统计代码块依赖 `DataviewJS`。

## 快速开始

1. 把本目录中的模板文件复制到你的 Obsidian vault 模板目录。
2. 在 Obsidian 中启用 `Templater` 和 `Dataview`。
3. 在 `Templater` 设置中指定模板目录。
4. 用日报模板新建每日笔记，再按周/月/季/年创建复盘笔记。
5. 保持日报 frontmatter 字段持续填写，后续汇总才有数据来源。

## 推荐命名

模板里的 DataviewJS 同时兼容两类数据来源：

- 文件名以日期开头，例如 `2026-03-20(...)`
- frontmatter 中有 `date:` 字段

建议至少保证以下两点：

- 日报文件名以 `YYYY-MM-DD` 开头
- 日报保留 `type`、`date`、`energy`、`mood`、`sleep_hours` 等字段

周报、月报、季报、年报模板也依赖文件名中的时间信息，建议使用：

- 周报：`2026-W12` 或 `2026-03(W12)`
- 月报：`2026-03`
- 季报：`2026-Q1`
- 年报：`2026`

## 自动汇总机制

这套模板的核心不是静态文本，而是“日报持续记录 + 高周期自动聚合”。

自动汇总主要统计这些信息：

- 记录天数
- 平均能量与情绪
- 睡眠时长
- 运动天数
- 阅读量
- 投资收益
- 任务完成情况

如果你希望周/月/季/年报能自动出数，日报里要尽量稳定填写 frontmatter 和关键区块。

## 适合的使用方式

- 每天用日报承接任务、习惯、阅读、投资和复盘
- 每周回顾执行质量和节奏问题
- 每月看趋势，不只看单日状态
- 每季度校正策略
- 每年做完整复盘

## 注意事项

- 文件名中的 `Dairy` 沿用了当前仓库已有命名；如果你以后改成 `Diary`，需要同步调整引用和自己的模板习惯
- DataviewJS 统计依赖你的实际字段命名，改字段前要一起改查询逻辑
- 模板默认包含较多个人管理和投资复盘内容，可以按自己的使用习惯删减

## 进一步阅读

- [00_Template_User_Guide.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/00_Template_User_Guide.md)
- [00_AutoAggregation_Scripts.md](/Users/armewang/Documents/CS-Tech/Local/obsidian日志模版/00_AutoAggregation_Scripts.md)
