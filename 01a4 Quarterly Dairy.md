---
type: quarterly-note
title: 📊 <% tp.file.title %>
date: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
quarter: <% tp.date.now("YYYY-[Q]Q") %>
year: <% tp.date.now("YYYY") %>
tags:
  - quarterlynote
  - review
  - strategy
energy_avg: /5
productivity_avg: /5
goal_completion: /100
---

# <% tp.file.title %> 季度报告

> [!quote] 本季度主题
>

---

# 📊 Quarter at a Glance

## 季度核心指标

| 维度 | Q目标 | Q实际 | 达成率 | vs上季 | 年度进度 |
|:-----|:-----:|:-----:|:------:|:------:|:--------:|
| 🎯 OKR 完成度 | 100% | __% | __% | ↑/↓ | __/100% |
| 💰 投资收益 | __% | __% | __% | ↑/↓ | __% YTD |
| 💵 储蓄金额 | ¥___ | ¥___ | __% | ↑/↓ | ¥___ YTD |
| 📚 阅读书籍 | __本 | __本 | __% | ↑/↓ | __本 YTD |
| 🏃 运动天数 | __天 | __天 | __% | ↑/↓ | __天 YTD |
| 📝 笔记产出 | __篇 | __篇 | __% | ↑/↓ | __篇 YTD |

## 📊 自动汇总 (来自日报)

```dataviewjs
// 获取本季度信息 - 支持格式: 2024-Q1
const quarterFile = dv.current().file.name;
const match = quarterFile.match(/(\d{4})-Q(\d)/);
if (match) {
  const year = parseInt(match[1]);
  const quarter = parseInt(match[2]);

  // 计算季度日期范围
  const startMonth = (quarter - 1) * 3 + 1;
  const endMonth = quarter * 3;
  const startDate = `${year}-${String(startMonth).padStart(2, '0')}-01`;
  const lastDay = new Date(year, endMonth, 0).getDate();
  const endDate = `${year}-${String(endMonth).padStart(2, '0')}-${lastDay}`;

  // 获取本季度所有日报 - 支持文件名格式: 2024-01-15(W03·Mon)
  const dailyNotes = dv.pages('#daily OR #dailynote OR #journal')
    .where(p => {
      const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
      if (dateMatch) {
        const fileDate = dateMatch[1];
        return fileDate >= startDate && fileDate <= endDate;
      }
      if (p.date) {
        const d = String(p.date).substring(0, 10);
        return d >= startDate && d <= endDate;
      }
      return false;
    });

  if (dailyNotes.length > 0) {
    const vals = dailyNotes.values;
    const totalDays = vals.length;

    dv.paragraph(`📊 **${year}年 Q${quarter}** 共收集 **${totalDays}** 天日报数据`);

    const stats = {
      avgEnergy: (vals.reduce((s, p) => s + (parseFloat(p.energy) || 0), 0) / totalDays).toFixed(1),
      avgMood: (vals.reduce((s, p) => s + (parseFloat(p.mood) || 0), 0) / totalDays).toFixed(1),
      exerciseDays: vals.filter(p => p.exercise === true || p.exercise === "true").length,
      totalReading: vals.reduce((s, p) => s + (parseFloat(p.reading_pages) || 0), 0),
      totalInvestment: vals.reduce((s, p) => s + (parseFloat(p.investment_return) || 0), 0).toFixed(2)
    };

    dv.table(
      ["维度", "季度数据", "月均", "评价"],
      [
        ["📝 记录天数", `${totalDays}天`, `${(totalDays/3).toFixed(0)}天/月`, totalDays >= 75 ? "✅" : "⚠️"],
        ["⚡ 平均能量", `${stats.avgEnergy}/5`, "-", stats.avgEnergy >= 3.5 ? "✅" : "⚠️"],
        ["😊 平均情绪", `${stats.avgMood}/5`, "-", stats.avgMood >= 3.5 ? "✅" : "⚠️"],
        ["🏃 运动天数", `${stats.exerciseDays}天`, `${(stats.exerciseDays/3).toFixed(0)}天/月`, stats.exerciseDays >= 60 ? "✅" : "⚠️"],
        ["📚 阅读总量", `${stats.totalReading}页`, `${(stats.totalReading/3).toFixed(0)}页/月`, stats.totalReading >= 900 ? "✅" : "⚠️"],
        ["💰 季度投资收益", `${stats.totalInvestment}%`, "-", stats.totalInvestment > 0 ? "✅ 盈利" : "⚠️ 亏损"]
      ]
    );

    // 月度对比
    dv.header(4, "📅 月度对比");
    const monthData = [0, 1, 2].map(i => {
      const m = startMonth + i;
      const mStart = `${year}-${String(m).padStart(2, '0')}-01`;
      const mLastDay = new Date(year, m, 0).getDate();
      const mEnd = `${year}-${String(m).padStart(2, '0')}-${mLastDay}`;
      const monthNotes = dailyNotes.where(p => {
        const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
        if (dateMatch) return dateMatch[1] >= mStart && dateMatch[1] <= mEnd;
        return false;
      });
      if (monthNotes.length === 0) return [`${m}月`, "-", "-", "-"];
      const mVals = monthNotes.values;
      return [
        `${m}月`,
        mVals.length + "天",
        (mVals.reduce((s, p) => s + (parseFloat(p.energy) || 0), 0) / mVals.length).toFixed(1),
        mVals.filter(p => p.exercise === true || p.exercise === "true").length + "天"
      ];
    });
    dv.table(["月份", "记录天数", "平均能量", "运动天数"], monthData);

  } else {
    dv.paragraph(`⚠️ ${year}年 Q${quarter} 暂无日报数据`);
    dv.paragraph("请确保日报包含 `#daily` 标签，且文件名以日期开头如 `2024-01-15(...)`");
  }
} else {
  dv.paragraph("⚠️ 文件名格式应为 YYYY-Q#（如 2024-Q1）");
}
```

## 季度情绪曲线

| 月份 | 能量 | 情绪 | 效率 | 关键词 |
|:----:|:----:|:----:|:----:|:-------|
| M1 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |
| M2 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |
| M3 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |

---

# 📅 月报汇总

```dataview
TABLE WITHOUT ID
  file.link AS "📝 月报",
  productivity AS "⚡ 效率",
  savings_rate AS "💰 储蓄率"
FROM #monthlynote
WHERE file.folder != "0-System"
  AND contains(file.name, "<% tp.date.now('YYYY') %>")
SORT file.name DESC
LIMIT 3
```

---

# 🎯 Quarterly OKR Review

## Objective 1: ___

> [!info] 目标描述
>

| Key Result | 目标值 | Q1 | Q2 | Q3 | Q4 | 状态 |
|:-----------|:------:|:--:|:--:|:--:|:--:|:----:|
| KR1: |  |  |  |  |  | 待定 |
| KR2: |  |  |  |  |  | 待定 |
| KR3: |  |  |  |  |  | 待定 |

**季度复盘：**
- 达成情况：
- 未达成原因：
- 经验教训：

## Objective 2: ___

| Key Result | 目标值 | Q1 | Q2 | Q3 | Q4 | 状态 |
|:-----------|:------:|:--:|:--:|:--:|:--:|:----:|
| KR1: |  |  |  |  |  | ⬜ |
| KR2: |  |  |  |  |  | ⬜ |

**季度复盘：**

## Objective 3: ___

| Key Result | 目标值 | Q1 | Q2 | Q3 | Q4 | 状态 |
|:-----------|:------:|:--:|:--:|:--:|:--:|:----:|
| KR1: |  |  |  |  |  | ⬜ |
| KR2: |  |  |  |  |  | ⬜ |

**季度复盘：**

---

# 📈 Investment Quarterly

> [!summary] 季度投资回顾
> - 📈 季度收益: __% | YTD: __%
> - 🏆 最佳交易:
> - 💔 最差交易:
> - ✅ 有效策略:
> - ❌ 失效策略:
> - 🎯 下季策略:

---

# 💰 Financial Quarterly

## 季度财务总结

| 指标 | Q目标 | Q实际 | 达成率 |
|:-----|:-----:|:-----:|:------:|
| 总收入 | ¥___ | ¥___ | __% |
| 总支出 | ¥___ | ¥___ | __% |
| 净储蓄 | ¥___ | ¥___ | __% |
| 储蓄率 | __% | __% | - |
| 被动收入 | ¥___ | ¥___ | __% |

## 资产负债快照

| 资产类别 | 季初 | 季末 | 变化 |
|:---------|:----:|:----:|:----:|
| 现金/活期 | ¥___ | ¥___ | ↑/↓ |
| 股票投资 | ¥___ | ¥___ | ↑/↓ |
| 基金投资 | ¥___ | ¥___ | ↑/↓ |
| 加密货币 | ¥___ | ¥___ | ↑/↓ |
| 其他资产 | ¥___ | ¥___ | ↑/↓ |
| **净资产** | ¥___ | ¥___ | **↑/↓** |

## 财务健康评估

- 🎯 财务自由进度: ___%
- 📈 净资产增长率: ___%
- 💰 被动收入覆盖率: ___%

---

# 🔄 Quarterly Deep Review

## 季度总结

> [!abstract] 一句话总结本季度
>

### 🏆 本季度五大成就
1.
2.
3.
4.
5.

### 📚 本季度阅读

| 书名 | 作者 | 评分 | 核心收获 |
|:-----|:-----|:----:|:---------|
|  |  | ⭐⭐⭐⭐⭐ |  |
|  |  | ⭐⭐⭐⭐⭐ |  |
|  |  | ⭐⭐⭐⭐⭐ |  |

### 💡 本季度最大洞察 (Top 3)
1.
2.
3.

### ⚠️ 本季度最大教训
>

## 生活轮评估

> [!example] 生活平衡轮 (1-10分)

| 领域 | 评分 | 本季进展 | 下季重点 |
|:-----|:----:|:---------|:---------|
| 💼 事业/工作 | /10 |  |  |
| 💰 财务 | /10 |  |  |
| 🏃 健康 | /10 |  |  |
| 👨‍👩‍👧‍👦 家庭/关系 | /10 |  |  |
| 💑 亲密关系 | /10 |  |  |
| 👥 社交/友谊 | /10 |  |  |
| 📚 学习成长 | /10 |  |  |
| 🎮 休闲娱乐 | /10 |  |  |
| 🧘 精神/内在 | /10 |  |  |
| 🌍 贡献/影响 | /10 |  |  |

**生活平衡度: __/100**

---

# 📅 Next Quarter Planning

## 🎯 下季度 OKR

### Objective 1: ___
- [ ] KR1:
- [ ] KR2:
- [ ] KR3:

### Objective 2: ___
- [ ] KR1:
- [ ] KR2:

### Objective 3: ___
- [ ] KR1:
- [ ] KR2:

## 📋 下季度重点项目

| 项目 | 目标 | 里程碑 | 资源需求 |
|:-----|:-----|:-------|:---------|
| 1. |  |  |  |
| 2. |  |  |  |
| 3. |  |  |  |

## 🔄 习惯系统调整

| 类型 | 习惯 | 原因 |
|:-----|:-----|:-----|
| ➕ 新增 |  |  |
| ⬆️ 强化 |  |  |
| ⬇️ 减少 |  |  |
| ❌ 删除 |  |  |

## 📆 季度重要日程

| 月份 | 日期 | 事项 | 准备 |
|:----:|:----:|:-----|:-----|
| M1 |  |  |  |
| M2 |  |  |  |
| M3 |  |  |  |

---

# 🙏 Gratitude & Reflection

> [!heart] 本季度感恩清单
> 1.
> 2.
> 3.
> 4.
> 5.

> [!success] 本季度亮点时刻 (Top 3)
> 1.
> 2.
> 3.

> [!note] 给自己的一封信
>

---

# 🔗 LINKS

## 时间导航

| ⬅️ 上季度 | 📅 本年 | ➡️ 下季度 |
|:---------:|:------:|:---------:|
| [[<% tp.date.now("YYYY-[Q]Q", -90) %>]] | [[<% tp.date.now("YYYY") %>]] | [[<% tp.date.now("YYYY-[Q]Q", 90) %>]] |

## 本季度月报

```dataviewjs
const q = dv.current().file.name;
const year = q.split('-')[0];
const quarter = parseInt(q.split('Q')[1]);
const months = [(quarter-1)*3+1, (quarter-1)*3+2, (quarter-1)*3+3];
const patterns = months.map(m => `${year}-${String(m).padStart(2,'0')}`);

const pages = dv.pages('#monthlynote')
  .where(p => patterns.some(pat => p.file.name.includes(pat)))
  .sort(p => p.file.name);
dv.list(pages.file.link);
```

---

*Created: <% tp.date.now("YYYY-MM-DD HH:mm") %> | Template v2.0*
