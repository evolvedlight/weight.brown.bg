---
toc: false
---

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 2rem 0 4rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 2rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  background: linear-gradient(30deg, var(--theme-foreground-focus), currentColor);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 90px;
  }
}

</style>

<div class="hero">
  <h1>Goodbye, excess weight</h1>
  <h2>Tracking a single person's weight loss</h2>
  <a href="https://brown.bg" target="_blank">back to the blog<span style="display: inline-block; margin-left: 0.25rem;">↗︎</span></a>
</div>

```js
function calculateRunningAverage(weights) {
  const averages = [];
  for (let i = 0; i < weights.length; i++) {
    let sum = 0;
    let count = 0;
    for (let j = i; j >= Math.max(0, i - 4); j--) {
      sum += weights[j].weight; // Add the weight property of the weight object
      count++;
    }
    const averageWeight = sum / count;
    const date = weights[i].date;
    averages.push({ averageWeight, date });
  }
  return averages;
}
const weights = await FileAttachment("./data/weight-mimi.csv").csv({typed: true});

const weightTable = [];
// add all weights to weight table
for (let i = 0; i < weights.length; i++) {
  const yesterdayTrendNumber = weightTable[i - 1]?.trend ?? weights[0].weight;
  
  // this is the 10% exponential smoothed moving average from the hacker's diet
  // Subtract yesterday's trend from today's weight. Write the result with a minus sign if it's negative.
  const diffFromYesterday = weights[i].weight - yesterdayTrendNumber;

  // Shift the decimal place in the resulting number one place to the left. Round the number to one decimal place by dropping the second decimal and increasing the first decimal by one if the second decimal place is 5 or greater.
  const shifted = evenRound(diffFromYesterday / 10, 1);
  const trend = yesterdayTrendNumber + shifted

  weightTable.push({
    date: weights[i].date,
    weight: weights[i].weight,
    diffFromYesterday: diffFromYesterday,
    shifted: shifted,
    trend: trend
  });
}

function evenRound(num, decimalPlaces) {
    var d = decimalPlaces || 0;
    var m = Math.pow(10, d);
    var n = +(d ? num * m : num).toFixed(8); // Avoid rounding errors
    var i = Math.floor(n), f = n - i;
    var e = 1e-8; // Allow for rounding errors in f
    var r = (f > 0.5 - e && f < 0.5 + e) ?
                ((i % 2 == 0) ? i : i + 1) : Math.round(n);
    return d ? r / m : r;
}

const lastEntryDate = new Date(weightTable[weightTable.length - 1].date);

const startDate = new Date(lastEntryDate);
startDate.setDate(startDate.getDate() - 40);

const weightTable60Days = weightTable.filter(entry => {
    const entryDate = new Date(entry.date);
    return entryDate >= startDate;
});
```

<div class="grid grid-cols-1" style="grid-auto-rows: 504px;">
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Weight and moving average (last 60 days)",
      width,
      grid: true,
      x: {label: "Date"},
      y: {label: "Body mass (kg)"},
      color: {domain: [-1, 0, 1], range: ["#4daf4a", "currentColor", "#e41a1c"]},
      marks: [
        Plot.dot(weightTable60Days, {y: "weight", x: "date", stroke: "green"}),
        Plot.lineY(weightTable60Days, {y: "trend", x: "date", stroke: "lightgreen", tip: true}),
        Plot.ruleX(weightTable60Days, {
          x: "date",
          y1: "weight",
          y2: "trend",
          stroke: (d) => Math.sign(d.weight - d.trend),
          strokeWidth: 4,
          strokeLinecap: "round"
        }),
      ],
      y: {domain: [71, 76]}
    }))
  }</div>
</div>

<div class="grid grid-cols-1" style="grid-auto-rows: 504px;">
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Weight and moving average",
      width,
      grid: true,
      x: {label: "Date"},
      y: {label: "Body mass (kg)"},
      color: {domain: [-1, 0, 1], range: ["#4daf4a", "currentColor", "#e41a1c"]},
      marks: [
        Plot.dot(weightTable, {y: "weight", x: "date", stroke: "green", tip: true}),
        Plot.lineY(weightTable, {y: "trend", x: "date", stroke: "lightgreen"}),
        Plot.ruleX(weightTable, {
          x: "date",
          y1: "weight",
          y2: "trend",
          stroke: (d) => Math.sign(d.weight - d.trend),
          strokeWidth: 4,
          strokeLinecap: "round"
        }),
        Plot.ruleY([76.6], {stroke: "yellow", strokeDasharray: "2,2"}),
      ],
      y: {domain: [70, 80]}
    }))
  }</div>
</div>

```js
const firstWeight = weightTable[0];
const lastWeight = weights[weights.length - 1];

const daysTracking = (lastWeight.date - firstWeight.date) / (1000 * 60 * 60 * 24);
const daysWithWeights = weightTable.length;
const goal = 68;

const currentWeightWeighted = weightTable[weightTable.length - 1].trend;
const currentWeightWeightedAgo = weightTable[weightTable.length - 11].trend

const weightLoss = firstWeight.weight - currentWeightWeighted;

const trend = currentWeightWeightedAgo - currentWeightWeighted;
const trendPerDay = trend / 10;

const trendOverall = (firstWeight.weight - currentWeightWeighted) / daysTracking;

const weightLossUntilGoal = currentWeightWeighted - goal;
const daysUntilGoal = weightLossUntilGoal / trendPerDay;

const expectedGoalDate = new Date(lastWeight.date.getTime() + daysUntilGoal * 24 * 60 * 60 * 1000);

const percentageDone = weightLoss / (firstWeight.weight - goal) * 100;

const goalNotReached = daysUntilGoal > 0;
const weightLossUntilGoalString = goalNotReached ? weightLossUntilGoal.toFixed(1) + "kg" : "🎉";
const daysUntilGoalString = goalNotReached ? daysUntilGoal.toFixed(0) : "🎉";

const expectedEndDateString = goalNotReached ? expectedGoalDate.toLocaleDateString('en-CH') : "🎉";
```
<div class="grid grid-cols-4">
  <div class="card">
    <h2>Start Weight</span></h2>
    <span class="big">${firstWeight.weight}kg</span>
  </div>
  <div class="card">
    <h2>Current Weight</span></h2>
    <span class="big">${currentWeightWeighted.toFixed(1)}kg</span>
  </div>
  <div class="card">
    <h2>Goal Weight</span></h2>
    <span class="big">${goal}kg</span>
  </div>
  <div class="card">
    <h2>Weight Loss so far</h2>
    <span class="big">${weightLoss.toFixed(1)}kg</span>
  </div>
  <div class="card">
    <h2>Weight loss until goal</h2>
    <span class="big">${weightLossUntilGoalString}</span>
  </div>
  <div class="card">
    <h2>Average per day (rolling trend)</h2>
    <span class="big">${trendPerDay.toFixed(2)}kg</span>
  </div>
  <div class="card">
    <h2>Days until goal</h2>
    <span class="big">${daysUntilGoalString}</span>
  </div>
  <div class="card">
    <h2>Expected goal end date</h2>
    <span class="big">${expectedEndDateString}</span>
  </div>
  <div class="card">
    <h2>% done</h2>
    <span class="big">${percentageDone.toFixed(0)}%</span>
  </div>
  <div class="card">
    <h2>Days tracking</h2>
    <span class="big">${daysTracking}</span>
  </div>
  <div class="card">
    <h2>Days weighed in</h2>
    <span class="big">${daysWithWeights}</span>
  </div>
  <div class="card">
    <h2>Average per day (overall)</h2>
    <span class="big">${trendOverall.toFixed(2)}kg</span>
  </div>
</div>