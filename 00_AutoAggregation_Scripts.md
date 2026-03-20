# Obsidian 自动汇总脚本集合

> [!info] 使用说明
> 这些脚本配合 Templater 插件使用，可以自动从日报数据生成周/月/季/年报的统计数据

---

## 前置要求

### 1. 日报 frontmatter 标准格式
确保每日日报包含以下可统计字段：

```yaml
---
type: daily-note
date: 2024-01-15
energy: 4        # 1-5
mood: 3          # 1-5
sleep_hours: 7.5
exercise: true   # true/false
reading_pages: 30
investment_return: 0.5  # 百分比
tasks_completed: 8
tasks_total: 10
---
```

---

## 周报自动汇总脚本

### 方法1: DataviewJS 实时计算（推荐）

在周报中插入以下代码块，会自动计算本周数据：

````markdown
```dataviewjs
// 获取本周日期范围
const weekNum = dv.current().file.name; // 假设文件名格式: 2024-W03
const year = weekNum.split('-W')[0];
const week = parseInt(weekNum.split('-W')[1]);

// 计算本周的起止日期
const jan1 = new Date(year, 0, 1);
const daysToMonday = (jan1.getDay() + 6) % 7;
const firstMonday = new Date(jan1);
firstMonday.setDate(jan1.getDate() - daysToMonday + (week - 1) * 7);
const startDate = firstMonday.toISOString().split('T')[0];
const endDate = new Date(firstMonday.getTime() + 6 * 24 * 60 * 60 * 1000).toISOString().split('T')[0];

// 获取本周所有日报
const dailyNotes = dv.pages('#daily OR #dailynote')
  .where(p => p.date >= startDate && p.date <= endDate);

if (dailyNotes.length > 0) {
  // 计算统计数据
  const stats = {
    count: dailyNotes.length,
    avgEnergy: (dailyNotes.values.reduce((s, p) => s + (p.energy || 0), 0) / dailyNotes.length).toFixed(1),
    avgMood: (dailyNotes.values.reduce((s, p) => s + (p.mood || 0), 0) / dailyNotes.length).toFixed(1),
    avgSleep: (dailyNotes.values.reduce((s, p) => s + (p.sleep_hours || 0), 0) / dailyNotes.length).toFixed(1),
    exerciseDays: dailyNotes.values.filter(p => p.exercise).length,
    totalReading: dailyNotes.values.reduce((s, p) => s + (p.reading_pages || 0), 0),
    tasksCompleted: dailyNotes.values.reduce((s, p) => s + (p.tasks_completed || 0), 0),
    tasksTotal: dailyNotes.values.reduce((s, p) => s + (p.tasks_total || 0), 0)
  };

  dv.header(3, "📊 本周自动汇总");
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
  dv.paragraph("⚠️ 本周暂无日报数据");
}
```
````

---

## 月报自动汇总脚本

在月报中插入以下代码块：

````markdown
```dataviewjs
// 获取本月日期范围
const monthStr = dv.current().file.name; // 假设文件名格式: 2024-01
const [year, month] = monthStr.split('-').map(Number);
const startDate = `${year}-${String(month).padStart(2, '0')}-01`;
const endDate = `${year}-${String(month).padStart(2, '0')}-31`;

// 获取本月所有日报
const dailyNotes = dv.pages('#daily OR #dailynote')
  .where(p => p.date >= startDate && p.date <= endDate);

// 获取本月所有周报
const weeklyNotes = dv.pages('#weeklynote')
  .where(p => p.file.name.startsWith(year.toString()));

if (dailyNotes.length > 0) {
  const stats = {
    count: dailyNotes.length,
    avgEnergy: (dailyNotes.values.reduce((s, p) => s + (p.energy || 0), 0) / dailyNotes.length).toFixed(1),
    avgMood: (dailyNotes.values.reduce((s, p) => s + (p.mood || 0), 0) / dailyNotes.length).toFixed(1),
    exerciseDays: dailyNotes.values.filter(p => p.exercise).length,
    totalReading: dailyNotes.values.reduce((s, p) => s + (p.reading_pages || 0), 0),
    avgInvestment: (dailyNotes.values.reduce((s, p) => s + (p.investment_return || 0), 0)).toFixed(2)
  };

  dv.header(3, "📊 本月自动汇总");
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

  // 显示能量/情绪趋势
  dv.header(4, "📈 能量趋势");
  const energyByWeek = {};
  dailyNotes.values.forEach(p => {
    const weekNum = `W${Math.ceil(parseInt(p.date.split('-')[2]) / 7)}`;
    if (!energyByWeek[weekNum]) energyByWeek[weekNum] = [];
    if (p.energy) energyByWeek[weekNum].push(p.energy);
  });

  const trendData = Object.entries(energyByWeek).map(([week, vals]) => [
    week,
    (vals.reduce((a,b) => a+b, 0) / vals.length).toFixed(1),
    "🔋".repeat(Math.round(vals.reduce((a,b) => a+b, 0) / vals.length))
  ]);
  dv.table(["周次", "平均能量", "可视化"], trendData);

} else {
  dv.paragraph("⚠️ 本月暂无日报数据");
}
```
````

---

## 季报自动汇总脚本

````markdown
```dataviewjs
// 获取本季度信息
const quarterStr = dv.current().file.name; // 假设文件名格式: 2024-Q1
const year = parseInt(quarterStr.split('-Q')[0]);
const quarter = parseInt(quarterStr.split('-Q')[1]);

// 计算季度日期范围
const startMonth = (quarter - 1) * 3 + 1;
const endMonth = quarter * 3;
const startDate = `${year}-${String(startMonth).padStart(2, '0')}-01`;
const endDate = `${year}-${String(endMonth).padStart(2, '0')}-31`;

// 获取本季度所有日报
const dailyNotes = dv.pages('#daily OR #dailynote')
  .where(p => p.date >= startDate && p.date <= endDate);

// 获取本季度月报
const monthlyNotes = dv.pages('#monthlynote')
  .where(p => {
    const m = parseInt(p.file.name.split('-')[1]);
    return p.file.name.startsWith(year.toString()) && m >= startMonth && m <= endMonth;
  });

if (dailyNotes.length > 0) {
  const totalDays = dailyNotes.length;

  dv.header(3, `📊 Q${quarter} 自动汇总 (${totalDays}天数据)`);

  const stats = {
    avgEnergy: (dailyNotes.values.reduce((s, p) => s + (p.energy || 0), 0) / totalDays).toFixed(1),
    avgMood: (dailyNotes.values.reduce((s, p) => s + (p.mood || 0), 0) / totalDays).toFixed(1),
    exerciseDays: dailyNotes.values.filter(p => p.exercise).length,
    totalReading: dailyNotes.values.reduce((s, p) => s + (p.reading_pages || 0), 0),
    totalInvestment: dailyNotes.values.reduce((s, p) => s + (p.investment_return || 0), 0).toFixed(2)
  };

  dv.table(
    ["维度", "季度数据", "月均", "评价"],
    [
      ["📝 记录天数", `${totalDays}天`, `${(totalDays/3).toFixed(0)}天/月`, totalDays >= 75 ? "✅" : "⚠️"],
      ["⚡ 平均能量", `${stats.avgEnergy}/5`, "-", stats.avgEnergy >= 3.5 ? "✅" : "⚠️"],
      ["🏃 运动天数", `${stats.exerciseDays}天`, `${(stats.exerciseDays/3).toFixed(0)}天/月`, stats.exerciseDays >= 60 ? "✅" : "⚠️"],
      ["📚 阅读总量", `${stats.totalReading}页`, `${(stats.totalReading/3).toFixed(0)}页/月`, stats.totalReading >= 900 ? "✅" : "⚠️"],
      ["💰 季度投资收益", `${stats.totalInvestment}%`, "-", stats.totalInvestment > 0 ? "✅ 盈利" : "⚠️ 亏损"]
    ]
  );

  // 月度对比
  dv.header(4, "📅 月度对比");
  const monthNames = [`${startMonth}月`, `${startMonth+1}月`, `${startMonth+2}月`];
  const monthData = monthNames.map((name, i) => {
    const m = startMonth + i;
    const monthNotes = dailyNotes.where(p => parseInt(p.date.split('-')[1]) === m);
    if (monthNotes.length === 0) return [name, "-", "-", "-"];
    return [
      name,
      monthNotes.length + "天",
      (monthNotes.values.reduce((s, p) => s + (p.energy || 0), 0) / monthNotes.length).toFixed(1),
      monthNotes.values.filter(p => p.exercise).length + "天"
    ];
  });
  dv.table(["月份", "记录天数", "平均能量", "运动天数"], monthData);

} else {
  dv.paragraph("⚠️ 本季度暂无日报数据");
}
```
````

---

## 年报自动汇总脚本

````markdown
```dataviewjs
// 获取年份
const yearStr = dv.current().file.name; // 假设文件名格式: 2024
const year = parseInt(yearStr);
const startDate = `${year}-01-01`;
const endDate = `${year}-12-31`;

// 获取全年日报
const dailyNotes = dv.pages('#daily OR #dailynote')
  .where(p => p.date >= startDate && p.date <= endDate);

if (dailyNotes.length > 0) {
  const totalDays = dailyNotes.length;

  dv.header(2, `📊 ${year}年 自动年度汇总`);
  dv.paragraph(`共收集 **${totalDays}** 天日报数据，覆盖率 **${(totalDays/365*100).toFixed(1)}%**`);

  const stats = {
    avgEnergy: (dailyNotes.values.reduce((s, p) => s + (p.energy || 0), 0) / totalDays).toFixed(2),
    avgMood: (dailyNotes.values.reduce((s, p) => s + (p.mood || 0), 0) / totalDays).toFixed(2),
    avgSleep: (dailyNotes.values.reduce((s, p) => s + (p.sleep_hours || 0), 0) / totalDays).toFixed(2),
    exerciseDays: dailyNotes.values.filter(p => p.exercise).length,
    totalReading: dailyNotes.values.reduce((s, p) => s + (p.reading_pages || 0), 0),
    totalInvestment: dailyNotes.values.reduce((s, p) => s + (p.investment_return || 0), 0).toFixed(2),
    tasksCompleted: dailyNotes.values.reduce((s, p) => s + (p.tasks_completed || 0), 0)
  };

  // 年度核心指标
  dv.header(3, "🎯 年度核心指标");
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
  dv.header(3, "📈 季度趋势对比");
  const quarterData = [1, 2, 3, 4].map(q => {
    const startM = (q - 1) * 3 + 1;
    const endM = q * 3;
    const qNotes = dailyNotes.where(p => {
      const m = parseInt(p.date.split('-')[1]);
      return m >= startM && m <= endM;
    });
    if (qNotes.length === 0) return [`Q${q}`, "-", "-", "-", "-"];
    return [
      `Q${q}`,
      qNotes.length + "天",
      (qNotes.values.reduce((s, p) => s + (p.energy || 0), 0) / qNotes.length).toFixed(1),
      qNotes.values.filter(p => p.exercise).length + "天",
      qNotes.values.reduce((s, p) => s + (p.investment_return || 0), 0).toFixed(2) + "%"
    ];
  });
  dv.table(["季度", "记录天数", "平均能量", "运动天数", "投资收益"], quarterData);

  // 月度热力图
  dv.header(3, "📅 月度记录热力图");
  const monthData = [];
  for (let m = 1; m <= 12; m++) {
    const monthNotes = dailyNotes.where(p => parseInt(p.date.split('-')[1]) === m);
    const count = monthNotes.length;
    const bar = "█".repeat(Math.round(count / 3));
    monthData.push([`${m}月`, count + "天", bar]);
  }
  dv.table(["月份", "记录天数", "可视化"], monthData);

  // 最佳/最差日
  dv.header(3, "🏆 年度高光时刻");
  const sortedByEnergy = dailyNotes.sort(p => p.energy, 'desc');
  const bestDays = sortedByEnergy.slice(0, 5);
  dv.paragraph("**能量最高的5天:**");
  dv.table(
    ["日期", "能量", "链接"],
    bestDays.values.map(p => [p.date, `${p.energy}/5`, p.file.link])
  );

} else {
  dv.paragraph("⚠️ 本年度暂无日报数据");
}
```
````

---

## 使用建议

### 1. 确保日报数据格式一致
frontmatter 中的字段名必须统一，如 `energy`, `mood`, `exercise` 等

### 2. 使用正确的文件命名
- 日报: `2024-01-15.md`
- 周报: `2024-W03.md`
- 月报: `2024-01.md`
- 季报: `2024-Q1.md`
- 年报: `2024.md`

### 3. 使用正确的标签
- 日报: `#daily` 或 `#dailynote`
- 周报: `#weeklynote`
- 月报: `#monthlynote`
- 季报: `#quarterlynote`
- 年报: `#yearlynote`

### 4. 安装必需插件
- **Dataview**: 必须安装，用于数据查询
- **Templater**: 推荐安装，用于模板变量

---

*Created: <% tp.date.now("YYYY-MM-DD HH:mm") %> | Auto-Aggregation Scripts v1.0*
