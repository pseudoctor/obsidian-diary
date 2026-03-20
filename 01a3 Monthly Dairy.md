---
type: monthly-note
title: 📆 <% tp.file.title %>
date: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
month: <% tp.date.now("YYYY-MM") %>
year: <% tp.date.now("YYYY") %>
tags:
  - monthlynote
  - review
energy_avg: /5
mood_avg: /5
productivity: /5
income:
expense:
savings_rate:
---

# <% tp.file.title %> 月报

> [!quote] 本月主题
>

---

# 📊 Month at a Glance

## 本月核心数据

| 维度 | 目标 | 实际 | 达成率 | vs上月 |
|:-----|:----:|:----:|:------:|:------:|
| 🎯 OKR 完成度 | 100% | __% | __% | ↑/↓ |
| 🏃 运动天数 | 20天 | __天 | __% | ↑/↓ |
| 📚 阅读书籍 | __本 | __本 | __% | ↑/↓ |
| 📝 笔记产出 | __篇 | __篇 | __% | ↑/↓ |
| 💰 投资收益 | - | __% | - | ↑/↓ |
| 💵 储蓄率 | __% | __% | __% | ↑/↓ |

## 📊 自动汇总 (来自日报)

```dataviewjs
// 获取本月日期范围 - 支持格式: 2024-01 或 2024-01 January
const monthFile = dv.current().file.name;
const match = monthFile.match(/^(\d{4})-(\d{2})/);
if (match) {
  const year = parseInt(match[1]);
  const month = parseInt(match[2]);
  const startDate = `${year}-${String(month).padStart(2, '0')}-01`;
  const lastDay = new Date(year, month, 0).getDate();
  const endDate = `${year}-${String(month).padStart(2, '0')}-${lastDay}`;

  // 获取本月所有日报 - 支持文件名格式: 2024-01-15(W03·Mon)
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
      exerciseDays: vals.filter(p => p.exercise === true || p.exercise === "true").length,
      totalReading: vals.reduce((s, p) => s + (parseFloat(p.reading_pages) || 0), 0),
      avgInvestment: (vals.reduce((s, p) => s + (parseFloat(p.investment_return) || 0), 0)).toFixed(2)
    };

    dv.paragraph(`📅 ${year}年${month}月 共 **${stats.count}** 天日报数据`);
    dv.table(
      ["指标", "本月数值", "日均", "评价"],
      [
        ["📝 记录天数", `${stats.count}天`, "-", stats.count >= 25 ? "✅ 优秀" : stats.count >= 20 ? "🟡 良好" : "⚠️ 需加强"],
        ["⚡ 能量状态", `平均 ${stats.avgEnergy}`, "-", stats.avgEnergy >= 3.5 ? "✅" : "⚠️"],
        ["😊 情绪状态", `平均 ${stats.avgMood}`, "-", stats.avgMood >= 3.5 ? "✅" : "⚠️"],
        ["🏃 运动天数", `${stats.exerciseDays}天`, `${(stats.exerciseDays/stats.count*100).toFixed(0)}%`, stats.exerciseDays >= 20 ? "✅" : "⚠️"],
        ["📚 阅读总量", `${stats.totalReading}页`, `${(stats.totalReading/stats.count).toFixed(0)}页/天`, stats.totalReading >= 300 ? "✅" : "⚠️"],
        ["💰 投资收益", `${stats.avgInvestment}%`, "-", stats.avgInvestment > 0 ? "✅" : "⚠️"]
      ]
    );

    // 周趋势 - 从文件名提取日期
    dv.header(4, "📈 周趋势");
    const weekData = [1, 2, 3, 4, 5].map(w => {
      const startDay = (w - 1) * 7 + 1;
      const endDay = Math.min(w * 7, lastDay);
      const weekNotes = dailyNotes.where(p => {
        const dateMatch = p.file.name.match(/^(\d{4})-(\d{2})-(\d{2})/);
        if (dateMatch) {
          const day = parseInt(dateMatch[3]);
          return day >= startDay && day <= endDay;
        }
        return false;
      });
      if (weekNotes.length === 0) return [`W${w}`, "-", "-", "-"];
      const wVals = weekNotes.values;
      const avgE = wVals.reduce((s, p) => s + (parseFloat(p.energy) || 0), 0) / wVals.length;
      return [
        `W${w}`,
        wVals.length + "天",
        avgE.toFixed(1),
        "🔋".repeat(Math.round(avgE))
      ];
    }).filter(row => row[1] !== "-");
    dv.table(["周次", "记录", "能量", "可视化"], weekData);

  } else {
    dv.paragraph(`⚠️ ${year}年${month}月暂无日报数据`);
    dv.paragraph("请确保日报包含 `#daily` 标签，且文件名以日期开头如 `2024-01-15(...)`");
  }
} else {
  dv.paragraph("⚠️ 文件名格式应以 YYYY-MM 开头（如 2024-01）");
}
```

## 本月情绪与能量曲线

| 周次 | 能量 | 情绪 | 效率 | 关键事件 |
|:----:|:----:|:----:|:----:|:---------|
| W1 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |
| W2 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |
| W3 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |
| W4 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |
| W5 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |

---

# 📅 周报汇总

```dataviewjs
// 从月报文件名解析年月 - 支持格式: 2024-01 或 2024-01 January
const monthFile = dv.current().file.name;
const match = monthFile.match(/^(\d{4})-(\d{2})/);

if (match) {
  const yearMonth = `${match[1]}-${match[2]}`;
  dv.paragraph(`📅 **${match[1]}年${parseInt(match[2])}月** 周报`);

  const pages = dv.pages('#weeklynote')
    .where(p => p.file.name.includes(yearMonth))
    .sort(p => p.file.name, 'desc');

  if (pages.length > 0) {
    dv.table(
      ["📝 周报", "⚡ 效率"],
      pages.map(p => [p.file.link, p.productivity || "-"])
    );
  } else {
    dv.paragraph("暂无本月周报数据");
  }
} else {
  dv.paragraph("⚠️ 文件名格式应以 YYYY-MM 开头（如 2024-01）");
}
```

---

# 🎯 OKR 月度回顾

## Objective 1: ___

| Key Result | 目标 | 实际 | 状态 |
|:-----------|:----:|:----:|:----:|
| KR1: | | | 待定 |
| KR2: | | | 待定 |
| KR3: | | | 待定 |

**达成分析：**

## Objective 2: ___

| Key Result | 目标 | 实际 | 状态 |
|:-----------|:----:|:----:|:----:|
| KR1: | | | 待定 |
| KR2: | | | 待定 |

**达成分析：**

## Objective 3: ___

| Key Result | 目标 | 实际 | 状态 |
|:-----------|:----:|:----:|:----:|
| KR1: | | | 待定 |
| KR2: | | | 待定 |

**达成分析：**

---

# 📈 Investment Monthly

> [!summary] 本月投资回顾
> - 📈 月收益: __% | YTD: __%
> - 🏆 最佳交易:
> - 💔 最差交易:
> - 📝 本月操作总结:
> - 💡 经验教训:
> - 🎯 下月策略:

---

# 💰 财务月报

## 收支概览

| 类别 | 预算 | 实际 | 差异 |
|:-----|:----:|:----:|:----:|
| 💵 收入 | ¥___ | ¥___ | ¥___ |
| 🏠 固定支出 | ¥___ | ¥___ | ¥___ |
| 🍽️ 餐饮 | ¥___ | ¥___ | ¥___ |
| 🚗 交通 | ¥___ | ¥___ | ¥___ |
| 🛒 购物 | ¥___ | ¥___ | ¥___ |
| 🎮 娱乐 | ¥___ | ¥___ | ¥___ |
| 📚 学习 | ¥___ | ¥___ | ¥___ |
| 其他 | ¥___ | ¥___ | ¥___ |
| **净储蓄** | ¥___ | ¥___ | ¥___ |

## 财务健康指标

- 储蓄率: ___%
- 被动收入: ¥___
- 财务自由进度: ___%

---

# 🔄 Monthly Review

## 本月小结

> [!abstract] 一句话总结本月
>

### 🏆 本月三大成就
1.
2.
3.

### 📚 本月阅读

| 书名 | 作者 | 评分 | 核心收获 |
|:-----|:-----|:----:|:---------|
|  |  | ⭐⭐⭐⭐⭐ |  |
|  |  | ⭐⭐⭐⭐⭐ |  |

### 💡 本月最大洞察
>

### ⚠️ 本月最大教训
>

## 生活各领域评估

| 领域 | 评分 | 本月进展 | 下月重点 |
|:-----|:----:|:---------|:---------|
| 💼 事业/工作 | ⭐⭐⭐⭐⭐ |  |  |
| 💰 财务 | ⭐⭐⭐⭐⭐ |  |  |
| 🏃 健康 | ⭐⭐⭐⭐⭐ |  |  |
| 👨‍👩‍👧‍👦 家庭/关系 | ⭐⭐⭐⭐⭐ |  |  |
| 📚 学习成长 | ⭐⭐⭐⭐⭐ |  |  |
| 🎮 休闲娱乐 | ⭐⭐⭐⭐⭐ |  |  |

---

# 📅 下月规划

## 🎯 下月 OKR

### Objective 1: ___
- [ ] KR1:
- [ ] KR2:
- [ ] KR3:

### Objective 2: ___
- [ ] KR1:
- [ ] KR2:

## 📋 下月重点任务

### 🔴 Must Do
- [ ]
- [ ]
- [ ]

### 🟡 Should Do
- [ ]
- [ ]

## 📆 重要日程

| 日期 | 事项 | 准备事项 |
|:----:|:-----|:---------|
|  |  |  |

## 习惯调整

| 习惯 | 调整 | 原因 |
|:-----|:-----|:-----|
| 新增 |  |  |
| 强化 |  |  |
| 减少 |  |  |

---

# 🙏 Gratitude & Highlights

> [!heart] 本月感恩清单
> 1.
> 2.
> 3.

> [!success] 本月亮点时刻
>

> [!note] 给自己的话
>

---

# 🔗 LINKS

## 时间导航

| ⬅️ 上月 | 📊 本季度 | 📅 本年 | ➡️ 下月 |
|:-------:|:--------:|:------:|:-------:|
| [[<% tp.date.now("YYYY-MM", -30) %>]] | [[<% tp.date.now("YYYY-[Q]Q") %>]] | [[<% tp.date.now("YYYY") %>]] | [[<% tp.date.now("YYYY-MM", 30) %>]] |

## 本月周报

```dataviewjs
const pages = dv.pages()
  .where(p => p.file.name.match(new RegExp(`${dv.current().file.name}-W\\d{2}`)))
  .sort(p => p.file.name);
dv.list(pages.file.link);
```

---

*Created: <% tp.date.now("YYYY-MM-DD HH:mm") %> | Template v2.0*
