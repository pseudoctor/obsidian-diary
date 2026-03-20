---
type: weekly-note
title: 🗓️ <% tp.file.title %>
date: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
week: <% tp.date.now("YYYY-[W]WW") %>
year: <% tp.date.now("YYYY") %>
tags:
  - weeklynote
  - review
energy_avg: /5
mood_avg: /5
productivity: /5
---

# <% tp.file.title %> 周报

> [!quote] 本周主题
>

---

# 📊 Week at a Glance

## 本周数据概览

| 指标 | 目标 | 实际 | 达成率 |
|:-----|:----:|:----:|:------:|
| 🎯 核心任务完成 | __ 项 | __ 项 | __% |
| 🏃 运动天数 | 5 天 | __ 天 | __% |
| 📚 阅读时长 | __ h | __ h | __% |
| 📝 笔记数量 | __ 篇 | __ 篇 | __% |
| 💰 投资收益 | - | __% | - |

## 📊 自动汇总 (来自日报)

```dataviewjs
// 获取本周信息 - 支持格式: 2024-W03 或 2024-01(W03)
const weekFile = dv.current().file.name;
let year, week;

// 尝试匹配 YYYY-W## 格式
const match1 = weekFile.match(/(\d{4})-W(\d{1,2})/);
// 尝试匹配 YYYY-MM(W##) 格式
const match2 = weekFile.match(/(\d{4})-\d{2}\(W(\d{1,2})/);

if (match1) {
  year = parseInt(match1[1]);
  week = parseInt(match1[2]);
} else if (match2) {
  year = parseInt(match2[1]);
  week = parseInt(match2[2]);
}

if (year && week) {
  // 计算本周的起止日期
  const jan1 = new Date(year, 0, 1);
  const daysToMonday = (jan1.getDay() + 6) % 7;
  const firstMonday = new Date(jan1);
  firstMonday.setDate(jan1.getDate() - daysToMonday + (week - 1) * 7);
  const startDate = firstMonday.toISOString().split('T')[0];
  const endDate = new Date(firstMonday.getTime() + 6 * 24 * 60 * 60 * 1000).toISOString().split('T')[0];

  // 获取本周所有日报 - 支持文件名格式: 2024-01-15(W03·Mon)
  const dailyNotes = dv.pages('#daily OR #dailynote OR #journal')
    .where(p => {
      // 从文件名提取日期 YYYY-MM-DD
      const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
      if (dateMatch) {
        const fileDate = dateMatch[1];
        return fileDate >= startDate && fileDate <= endDate;
      }
      // 或从 frontmatter date 字段获取
      if (p.date) {
        const d = String(p.date).substring(0, 10);
        return d >= startDate && d <= endDate;
      }
      return false;
    });

  if (dailyNotes.length > 0) {
    const vals = dailyNotes.values;
    const stats = {
      count: vals.length,
      avgEnergy: (vals.reduce((s, p) => s + (parseFloat(p.energy) || 0), 0) / vals.length).toFixed(1),
      avgMood: (vals.reduce((s, p) => s + (parseFloat(p.mood) || 0), 0) / vals.length).toFixed(1),
      avgSleep: (vals.reduce((s, p) => s + (parseFloat(p.sleep_hours) || 0), 0) / vals.length).toFixed(1),
      exerciseDays: vals.filter(p => p.exercise === true || p.exercise === "true").length,
      totalReading: vals.reduce((s, p) => s + (parseFloat(p.reading_pages) || 0), 0),
      tasksCompleted: vals.reduce((s, p) => s + (parseInt(p.tasks_completed) || 0), 0),
      tasksTotal: vals.reduce((s, p) => s + (parseInt(p.tasks_total) || 0), 0)
    };

    dv.paragraph(`📅 W${week} (${startDate} ~ ${endDate})`);
    dv.table(
      ["指标", "数值", "评价"],
      [
        ["📝 记录天数", `${stats.count}/7`, stats.count >= 5 ? "✅" : "⚠️"],
        ["⚡ 平均能量", `${stats.avgEnergy}/5`, stats.avgEnergy >= 3.5 ? "✅" : "⚠️"],
        ["😊 平均情绪", `${stats.avgMood}/5`, stats.avgMood >= 3.5 ? "✅" : "⚠️"],
        ["😴 平均睡眠", `${stats.avgSleep}h`, stats.avgSleep >= 7 ? "✅" : "⚠️"],
        ["🏃 运动天数", `${stats.exerciseDays}/7`, stats.exerciseDays >= 4 ? "✅" : "⚠️"],
        ["📚 阅读页数", `${stats.totalReading}页`, stats.totalReading >= 100 ? "✅" : "⚠️"],
        ["✅ 任务完成", `${stats.tasksCompleted}/${stats.tasksTotal}`,
         stats.tasksTotal > 0 ? `${(stats.tasksCompleted/stats.tasksTotal*100).toFixed(0)}%` : "-"]
      ]
    );
  } else {
    dv.paragraph(`⚠️ W${week} (${startDate} ~ ${endDate}) 暂无日报数据`);
    dv.paragraph("请确保日报包含 `#daily` 标签，且文件名以日期开头如 `2024-01-15(...)`");
  }
} else {
  dv.paragraph("⚠️ 无法解析周数，请确保文件名包含 W# 格式（如 2024-W03 或 2024-01(W03)）");
}
```

# 📅 日志汇总

> [!tip] 鼠标悬停于日志链接可激活 Hover Editor，拖动自动固定

```dataviewjs
// 从周报文件名解析周数 - 支持格式: 2024-W03 或 2024-01(W03)
const weekFile = dv.current().file.name;
let year, week;

const match1 = weekFile.match(/(\d{4})-W(\d{1,2})/);
const match2 = weekFile.match(/(\d{4})-\d{2}\(W(\d{1,2})/);

if (match1) {
  year = parseInt(match1[1]);
  week = parseInt(match1[2]);
} else if (match2) {
  year = parseInt(match2[1]);
  week = parseInt(match2[2]);
}

if (year && week) {
  // 计算该周的起止日期 (周一到周日)
  const jan1 = new Date(year, 0, 1);
  const daysToMonday = (jan1.getDay() + 6) % 7;
  const firstMonday = new Date(jan1);
  firstMonday.setDate(jan1.getDate() - daysToMonday + (week - 1) * 7);
  const startDate = firstMonday.toISOString().split('T')[0];
  const endDate = new Date(firstMonday.getTime() + 6 * 24 * 60 * 60 * 1000).toISOString().split('T')[0];

  dv.paragraph(`📅 **${startDate}** ~ **${endDate}**`);

  // 获取该周的日报
  const dailyNotes = dv.pages('#daily OR #dailynote OR #journal')
    .where(p => {
      const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
      if (dateMatch) {
        const fileDate = dateMatch[1];
        return fileDate >= startDate && fileDate <= endDate;
      }
      return false;
    })
    .sort(p => p.file.name, 'asc');

  if (dailyNotes.length > 0) {
    dv.table(
      ["📝 日记", "📅 日期"],
      dailyNotes.map(p => {
        const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
        return [p.file.link, dateMatch ? dateMatch[1] : p.file.name];
      })
    );
  } else {
    dv.paragraph("暂无本周日报数据");
  }
} else {
  dv.paragraph("⚠️ 无法解析周数，请确保文件名包含 W# 格式（如 2024-W03 或 2024-01(W03)）");
}
```

---

# ✅ 本周完成事项

## 🎯 核心目标达成

| 目标 | 状态 | 成果 | 备注 |
|:-----|:----:|:-----|:-----|
| 1. |  |  |  |
| 2. |  |  |  |
| 3. |  |  |  |

## 📋 任务完成清单

### 工作/学习
- [x]
- [x]
- [ ] ❌ 未完成:

### 个人项目
- [x]
- [x]

### 其他
- [x]

---

# 📈 Investment Weekly

> [!summary] 本周投资回顾
> - 📈 周收益: __%
> - 🏆 最佳表现:
> - 💔 最差表现:
> - 📝 本周操作:
> - 💡 经验教训:

---

# 🔄 Weekly Review

## 本周小结

> [!abstract] 一句话总结本周
>

### 🏆 本周三大成就
1.
2.
3.

### 📚 本周学到的
-

### 💡 本周洞察
-

### ⚠️ 本周教训
-

## SWOT 周复盘

| 内部因素 | 外部因素 |
|:---------|:---------|
| **S 做得好的** | **O 发现的机会** |
|  |  |
| **W 需改进的** | **T 面临的挑战** |
|  |  |

---

# 📅 下周规划

## 🎯 下周核心目标 (OKR)

> [!goal] 本周必须完成的 3 件事
> 1. 🐸 **MIT**:
> 2.
> 3.

## 📋 下周任务清单

### 🔴 Must Do (必须完成)
- [ ]
- [ ]
- [ ]

### 🟡 Should Do (应该完成)
- [ ]
- [ ]

### 🟢 Nice to Do (可以完成)
- [ ]
- [ ]

## ⏰ 重要日程

| 日期 | 时间 | 事项 | 准备 |
|:----:|:----:|:-----|:-----|
|  |  |  |  |

---

# 🙏 Gratitude & Highlights

> [!heart] 本周感恩
> - 感谢的人:
> - 感谢的事:

> [!success] 本周亮点时刻
>

---

# 🔗 LINKS

## 时间导航

| ⬅️ 上周 | 📆 本月 | 📊 本季度 | ➡️ 下周 |
|:-------:|:------:|:--------:|:-------:|
| [[<% tp.date.now("YYYY-[W]WW", -7) %>]] | [[<% tp.date.now("YYYY-MM") %>]] | [[<% tp.date.now("YYYY-[Q]Q") %>]] | [[<% tp.date.now("YYYY-[W]WW", 7) %>]] |

## 本周日记快速访问

```dataviewjs
const pages = dv.pages(`"${dv.current().file.folder}"`)
  .where(p => p.file.name.match(new RegExp(`${dv.current().file.name.split('-')[0]}-\\d{2}-\\d{2}`)))
  .sort(p => p.file.name);
dv.list(pages.file.link);
```

---

*Created: <% tp.date.now("YYYY-MM-DD HH:mm") %> | Template v2.0*
