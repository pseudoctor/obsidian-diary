---
type: yearly-note
title: 🎊 <% tp.file.title %>
date: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
year: <% tp.date.now("YYYY") %>
tags:
  - yearlynote
  - annual-review
  - strategy
theme:
word_of_year:
energy_avg: /5
happiness_avg: /5
growth_score: /10
---

# <% tp.file.title %> 年度报告

> [!quote] 年度主题词
> **Theme:**
> **Word of the Year:**

---

# 📊 Year at a Glance

## 年度核心成果

| 维度 | 年初目标 | 年末实际 | 达成率 | vs去年 |
|:-----|:--------:|:--------:|:------:|:------:|
| 🎯 年度目标完成 | __项 | __项 | __% | ↑/↓ |
| 💰 投资总收益 | __% | __% | - | ↑/↓ |
| 💵 净资产增长 | ¥___ | ¥___ | __% | ↑/↓ |
| 📚 阅读书籍 | __本 | __本 | __% | ↑/↓ |
| 🏃 运动天数 | __天 | __天 | __% | ↑/↓ |
| 📝 笔记/文章 | __篇 | __篇 | __% | ↑/↓ |
| ✈️ 旅行目的地 | __个 | __个 | __% | ↑/↓ |
| 🎓 技能/认证 | __项 | __项 | __% | ↑/↓ |

## 📊 自动年度汇总 (来自日报)

```dataviewjs
// 获取年份 - 支持格式: 2024 或 2024年
const yearFile = dv.current().file.name;
const match = yearFile.match(/^(\d{4})/);
if (match) {
  const year = parseInt(match[1]);
  const startDate = `${year}-01-01`;
  const endDate = `${year}-12-31`;

  // 获取全年日报 - 支持文件名格式: 2024-01-15(W03·Mon)
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

    dv.paragraph(`📅 **${year}年** 共收集 **${totalDays}** 天日报数据，覆盖率 **${(totalDays/365*100).toFixed(1)}%**`);

    const stats = {
      avgEnergy: (vals.reduce((s, p) => s + (parseFloat(p.energy) || 0), 0) / totalDays).toFixed(2),
      avgMood: (vals.reduce((s, p) => s + (parseFloat(p.mood) || 0), 0) / totalDays).toFixed(2),
      avgSleep: (vals.reduce((s, p) => s + (parseFloat(p.sleep_hours) || 0), 0) / totalDays).toFixed(2),
      exerciseDays: vals.filter(p => p.exercise === true || p.exercise === "true").length,
      totalReading: vals.reduce((s, p) => s + (parseFloat(p.reading_pages) || 0), 0),
      totalInvestment: vals.reduce((s, p) => s + (parseFloat(p.investment_return) || 0), 0).toFixed(2),
      tasksCompleted: vals.reduce((s, p) => s + (parseInt(p.tasks_completed) || 0), 0)
    };

    // 年度核心指标
    dv.header(4, "🎯 年度核心指标");
    dv.table(
      ["指标", "年度数据", "日均/率", "评价"],
      [
        ["📝 日报记录", `${totalDays}天`, `${(totalDays/365*100).toFixed(1)}%覆盖`, totalDays >= 300 ? "🏆 优秀" : totalDays >= 200 ? "✅ 良好" : "⚠️ 需加强"],
        ["⚡ 能量状态", `平均 ${stats.avgEnergy}/5`, "-", stats.avgEnergy >= 3.5 ? "✅" : "⚠️"],
        ["😊 情绪状态", `平均 ${stats.avgMood}/5`, "-", stats.avgMood >= 3.5 ? "✅" : "⚠️"],
        ["😴 睡眠时长", `平均 ${stats.avgSleep}h`, "-", stats.avgSleep >= 7 ? "✅" : "⚠️"],
        ["🏃 运动天数", `${stats.exerciseDays}天`, `${(stats.exerciseDays/totalDays*100).toFixed(1)}%`, stats.exerciseDays >= 200 ? "🏆" : stats.exerciseDays >= 150 ? "✅" : "⚠️"],
        ["📚 阅读总量", `${stats.totalReading}页`, `${(stats.totalReading/totalDays).toFixed(1)}页/天`, stats.totalReading >= 3000 ? "🏆" : "⚠️"],
        ["💰 年度投资", `${stats.totalInvestment}%`, "-", stats.totalInvestment > 10 ? "🏆" : stats.totalInvestment > 0 ? "✅" : "⚠️"],
        ["✅ 任务完成", `${stats.tasksCompleted}项`, `${(stats.tasksCompleted/totalDays).toFixed(1)}项/天`, "-"]
      ]
    );

    // 季度对比
    dv.header(4, "📈 季度趋势对比");
    const quarterData = [1, 2, 3, 4].map(q => {
      const startM = (q - 1) * 3 + 1;
      const endM = q * 3;
      const qStart = `${year}-${String(startM).padStart(2, '0')}-01`;
      const qLastDay = new Date(year, endM, 0).getDate();
      const qEnd = `${year}-${String(endM).padStart(2, '0')}-${qLastDay}`;
      const qNotes = dailyNotes.where(p => {
        const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
        if (dateMatch) return dateMatch[1] >= qStart && dateMatch[1] <= qEnd;
        return false;
      });
      if (qNotes.length === 0) return [`Q${q}`, "-", "-", "-", "-"];
      const qVals = qNotes.values;
      return [
        `Q${q}`,
        qVals.length + "天",
        (qVals.reduce((s, p) => s + (parseFloat(p.energy) || 0), 0) / qVals.length).toFixed(1),
        qVals.filter(p => p.exercise === true || p.exercise === "true").length + "天",
        qVals.reduce((s, p) => s + (parseFloat(p.investment_return) || 0), 0).toFixed(2) + "%"
      ];
    });
    dv.table(["季度", "记录天数", "平均能量", "运动天数", "投资收益"], quarterData);

    // 月度热力图
    dv.header(4, "📅 月度记录热力图");
    const monthData = [];
    for (let m = 1; m <= 12; m++) {
      const mStart = `${year}-${String(m).padStart(2, '0')}-01`;
      const mLastDay = new Date(year, m, 0).getDate();
      const mEnd = `${year}-${String(m).padStart(2, '0')}-${mLastDay}`;
      const monthNotes = dailyNotes.where(p => {
        const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
        if (dateMatch) return dateMatch[1] >= mStart && dateMatch[1] <= mEnd;
        return false;
      });
      const count = monthNotes.length;
      const bar = "█".repeat(Math.round(count / 3));
      monthData.push([`${m}月`, count + "天", bar]);
    }
    dv.table(["月份", "记录天数", "可视化"], monthData);

    // 能量最高的5天
    dv.header(4, "🏆 年度高光时刻（能量最高的5天）");
    const sortedByEnergy = dailyNotes.sort(p => parseFloat(p.energy) || 0, 'desc').slice(0, 5);
    dv.table(
      ["日期", "能量", "链接"],
      sortedByEnergy.values.map(p => {
        const dateMatch = p.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
        return [dateMatch ? dateMatch[1] : p.file.name, `${p.energy}/5`, p.file.link];
      })
    );

  } else {
    dv.paragraph(`⚠️ ${year}年暂无日报数据`);
    dv.paragraph("请确保日报包含 `#daily` 标签，且文件名以日期开头如 `2024-01-15(...)`");
  }
} else {
  dv.paragraph("⚠️ 文件名应以年份开头（如 2024 或 2024年）");
}
```

## 年度情绪全景图

| 季度 | 能量 | 情绪 | 效率 | 关键词 | 代表事件 |
|:----:|:----:|:----:|:----:|:-------|:---------|
| Q1 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |  |
| Q2 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |  |
| Q3 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |  |
| Q4 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |  |  |

---

# 📅 季度报告汇总

```dataview
TABLE WITHOUT ID
  file.link AS "📊 季报",
  goal_completion AS "🎯 目标完成度",
  productivity_avg AS "⚡ 效率"
FROM #quarterlynote
WHERE contains(file.name, "<% tp.date.now('YYYY') %>")
SORT file.name ASC
```

---

# 🎯 Annual OKR Review

## Objective 1: ___

> [!info] 目标描述
>

| Key Result | 目标 | Q1 | Q2 | Q3 | Q4 | 年度 | 状态 |
|:-----------|:----:|:--:|:--:|:--:|:--:|:----:|:----:|
| KR1: |  |  |  |  |  |  | 待定 |
| KR2: |  |  |  |  |  |  | 待定 |
| KR3: |  |  |  |  |  |  | 待定 |

**年度复盘：**
- 达成情况：
- 关键成功因素：
- 未达成原因：
- 经验教训：

## Objective 2: ___

| Key Result | 目标 | Q1 | Q2 | Q3 | Q4 | 年度 | 状态 |
|:-----------|:----:|:--:|:--:|:--:|:--:|:----:|:----:|
| KR1: |  |  |  |  |  |  | ⬜ |
| KR2: |  |  |  |  |  |  | ⬜ |

**年度复盘：**

## Objective 3: ___

| Key Result | 目标 | Q1 | Q2 | Q3 | Q4 | 年度 | 状态 |
|:-----------|:----:|:--:|:--:|:--:|:--:|:----:|:----:|
| KR1: |  |  |  |  |  |  | ⬜ |
| KR2: |  |  |  |  |  |  | ⬜ |

**年度复盘：**

---

# 📈 Investment Annual Review

> [!summary] 年度投资回顾
> - 📈 年度收益: __%
> - 📊 vs 基准: __%
> - 🏆 年度最佳交易:
> - 💔 年度最差交易:
> - ✅ 有效的投资策略:
> - ❌ 失效的投资策略:
> - 🎯 明年投资策略:

---

# 💰 Financial Annual Report

## 年度财务总结

| 指标 | 年初目标 | 年末实际 | 达成率 | vs去年 |
|:-----|:--------:|:--------:|:------:|:------:|
| 总收入 | ¥___ | ¥___ | __% | ↑/↓ __% |
| 总支出 | ¥___ | ¥___ | __% | ↑/↓ __% |
| 净储蓄 | ¥___ | ¥___ | __% | ↑/↓ __% |
| 储蓄率 | __% | __% | - | ↑/↓ |
| 被动收入 | ¥___ | ¥___ | __% | ↑/↓ __% |
| 投资收益 | ¥___ | ¥___ | __% | ↑/↓ __% |

## 年度资产负债表

| 资产类别 | 年初 | 年末 | 变化 | 占比 |
|:---------|:----:|:----:|:----:|:----:|
| 💵 现金/活期 | ¥___ | ¥___ | ↑/↓ | __% |
| 📈 股票投资 | ¥___ | ¥___ | ↑/↓ | __% |
| 📊 基金投资 | ¥___ | ¥___ | ↑/↓ | __% |
| 🪙 加密货币 | ¥___ | ¥___ | ↑/↓ | __% |
| 🏠 房产 | ¥___ | ¥___ | ↑/↓ | __% |
| 🚗 其他资产 | ¥___ | ¥___ | ↑/↓ | __% |
| ➖ 负债 | ¥___ | ¥___ | ↑/↓ | - |
| **🏦 净资产** | ¥___ | ¥___ | **↑/↓ __%** | 100% |

## 财务自由进度

| 指标 | 当前 | 目标 | 进度 |
|:-----|:----:|:----:|:----:|
| 净资产 | ¥___ | ¥___ | __% |
| 被动收入/月 | ¥___ | ¥___ | __% |
| 被动收入覆盖率 | __% | 100% | __% |
| 预计达成年份 | - | 20__ | - |

---

# 🔄 Annual Deep Review

## 年度总结

> [!abstract] 一句话总结这一年
>

### 🏆 年度十大成就
1.
2.
3.
4.
5.
6.
7.
8.
9.
10.

### 📚 年度阅读清单

| # | 书名 | 作者 | 评分 | 核心收获 |
|:-:|:-----|:-----|:----:|:---------|
| 1 |  |  | ⭐⭐⭐⭐⭐ |  |
| 2 |  |  | ⭐⭐⭐⭐⭐ |  |
| 3 |  |  | ⭐⭐⭐⭐⭐ |  |
| 4 |  |  | ⭐⭐⭐⭐⭐ |  |
| 5 |  |  | ⭐⭐⭐⭐⭐ |  |

**年度最佳书籍：**

### 💡 年度最大洞察 (Top 5)
1.
2.
3.
4.
5.

### ⚠️ 年度最大教训 (Top 3)
1.
2.
3.

### 🎭 年度关键词
> 用3-5个词概括这一年

---

## 生活轮年度评估

> [!example] 生活平衡轮 (1-10分)

| 领域 | 年初 | 年末 | 变化 | 明年目标 |
|:-----|:----:|:----:|:----:|:--------:|
| 💼 事业/工作 | /10 | /10 | ↑/↓ | /10 |
| 💰 财务 | /10 | /10 | ↑/↓ | /10 |
| 🏃 健康 | /10 | /10 | ↑/↓ | /10 |
| 👨‍👩‍👧‍👦 家庭 | /10 | /10 | ↑/↓ | /10 |
| 💑 亲密关系 | /10 | /10 | ↑/↓ | /10 |
| 👥 社交/友谊 | /10 | /10 | ↑/↓ | /10 |
| 📚 学习成长 | /10 | /10 | ↑/↓ | /10 |
| 🎮 休闲娱乐 | /10 | /10 | ↑/↓ | /10 |
| 🧘 精神/内在 | /10 | /10 | ↑/↓ | /10 |
| 🌍 贡献/影响 | /10 | /10 | ↑/↓ | /10 |

**年度生活满意度: __/100**

---

## 年度里程碑事件

| 月份 | 事件 | 类型 | 影响 |
|:----:|:-----|:----:|:-----|
| 1月 |  |  |  |
| 2月 |  |  |  |
| 3月 |  |  |  |
| 4月 |  |  |  |
| 5月 |  |  |  |
| 6月 |  |  |  |
| 7月 |  |  |  |
| 8月 |  |  |  |
| 9月 |  |  |  |
| 10月 |  |  |  |
| 11月 |  |  |  |
| 12月 |  |  |  |

---

# 📅 Next Year Planning

## 🎯 明年愿景

> [!goal] 明年主题
> **Theme:**
> **Word of the Year:**

## 明年三大核心目标

### 🥇 Goal 1: ___

| Key Result | 目标 | 衡量方式 |
|:-----------|:----:|:---------|
| KR1: |  |  |
| KR2: |  |  |
| KR3: |  |  |

### 🥈 Goal 2: ___

| Key Result | 目标 | 衡量方式 |
|:-----------|:----:|:---------|
| KR1: |  |  |
| KR2: |  |  |

### 🥉 Goal 3: ___

| Key Result | 目标 | 衡量方式 |
|:-----------|:----:|:---------|
| KR1: |  |  |
| KR2: |  |  |

## 📋 明年重点项目

| 项目 | 目标 | 时间线 | 关键里程碑 |
|:-----|:-----|:------:|:-----------|
| 1. |  | Q_ |  |
| 2. |  | Q_ |  |
| 3. |  | Q_ |  |

## 🔄 习惯系统年度调整

| 类型 | 习惯 | 频率 | 原因 |
|:-----|:-----|:----:|:-----|
| ➕ 新增 |  |  |  |
| ⬆️ 强化 |  |  |  |
| ⬇️ 减少 |  |  |  |
| ❌ 删除 |  |  |  |

## 📆 明年重要日程

| 季度 | 事项 | 日期 | 准备 |
|:----:|:-----|:----:|:-----|
| Q1 |  |  |  |
| Q2 |  |  |  |
| Q3 |  |  |  |
| Q4 |  |  |  |

## 💰 明年财务目标

| 指标 | 今年实际 | 明年目标 | 增长率 |
|:-----|:--------:|:--------:|:------:|
| 总收入 | ¥___ | ¥___ | __% |
| 储蓄率 | __% | __% | - |
| 被动收入 | ¥___ | ¥___ | __% |
| 净资产 | ¥___ | ¥___ | __% |

---

# 🙏 Gratitude & Legacy

> [!heart] 年度感恩清单
>
> **感谢的人：**
> 1.
> 2.
> 3.
> 4.
> 5.
>
> **感谢的事：**
> 1.
> 2.
> 3.

> [!success] 年度亮点时刻 (Top 5)
> 1.
> 2.
> 3.
> 4.
> 5.

> [!note] 给明年自己的一封信
>
> 亲爱的明年的我，
>
>
>
> —— <% tp.date.now("YYYY") %>年的我

---

# 🔗 LINKS

## 时间导航

| ⬅️ 去年 | ➡️ 明年 |
|:-------:|:-------:|
| [[<% tp.date.now("YYYY", -365) %>]] | [[<% tp.date.now("YYYY", 365) %>]] |

## 本年度季报

```dataviewjs
// 从年报文件名解析年份 - 支持格式: 2024 或 2024年
const yearFile = dv.current().file.name;
const match = yearFile.match(/^(\d{4})/);

if (match) {
  const year = match[1];
  const pages = dv.pages('#quarterlynote')
    .where(p => p.file.name.includes(year))
    .sort(p => p.file.name);

  if (pages.length > 0) {
    dv.list(pages.file.link);
  } else {
    dv.paragraph("暂无本年度季报数据");
  }
} else {
  dv.paragraph("⚠️ 文件名应以年份开头（如 2024）");
}
```

## 本年度月报

```dataviewjs
// 从年报文件名解析年份
const yearFile = dv.current().file.name;
const match = yearFile.match(/^(\d{4})/);

if (match) {
  const year = match[1];
  const pages = dv.pages('#monthlynote')
    .where(p => p.file.name.includes(year))
    .sort(p => p.file.name);

  if (pages.length > 0) {
    dv.list(pages.file.link);
  } else {
    dv.paragraph("暂无本年度月报数据");
  }
} else {
  dv.paragraph("⚠️ 文件名应以年份开头（如 2024）");
}
```

---

*Created: <% tp.date.now("YYYY-MM-DD HH:mm") %> | Template v2.0*
