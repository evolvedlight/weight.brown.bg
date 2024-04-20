---
toc: false
---

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 8rem;
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
const weights = await FileAttachment("./data/weight.csv").csv({typed: true});
const rollingWeight = calculateRunningAverage(weights);
```

<div class="grid grid-cols-1" style="grid-auto-rows: 504px;">
  <div class="card">${
    resize((width) => Plot.plot({
      title: "Weight and 5 day rolling average 🐧",
      width,
      grid: true,
      x: {label: "Date"},
      y: {label: "Body mass (kg)"},
      marks: [
        Plot.dot(weights, {y: "weight", x: "date", stroke: "green", tip: true}),
        Plot.lineY(rollingWeight, {y: "averageWeight", x: "date", stroke: "lightgreen"})
      ],
      y: {domain: [80, 88]}
    }))
  }</div>
</div>

```js
const firstWeight = weights[0];
const lastWeight = weights[weights.length - 1];
const daysTracking = (lastWeight.date - firstWeight.date) / (1000 * 60 * 60 * 24);
const goal = 75;

// for the rolling average we just weight naively the last 5 rows
var last5 = weights.slice(-5);
var last5Ago = weights.slice(-10, -5);
const currentWeightWeighted = calculateWeightedAverage(last5)
const currentWeightWeightedAgo = calculateWeightedAverage(last5Ago)
const weightLoss = firstWeight.weight - currentWeightWeighted;

const trend = currentWeightWeightedAgo - currentWeightWeighted;
const trendPerDay = trend / 5;
const weightLossUntilGoal = currentWeightWeighted - goal;
const daysUntilGoal = weightLossUntilGoal / trendPerDay;

function calculateWeightedAverage(weights) {
  const smoothingFactor = 2 / (weights.length + 1);
  let ema = weights[0].weight; // Start with the first day's weight

  for (let i = 1; i < weights.length; i++) {
    ema = (weights[i].weight * smoothingFactor) + (ema * (1 - smoothingFactor));
  }

  return ema;
}
```
<div class="grid grid-cols-3">
  <div class="card">
    <h2>Start Weight</span></h2>
    <span class="big">${firstWeight.weight}kg</span>
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
    <h2>Average per day (rolling trend)</h2>
    <span class="big">${trendPerDay.toFixed(2)}kg</span>
  </div>
  <div class="card">
    <h2>Days until goal</h2>
    <span class="big">${daysUntilGoal.toFixed(0)}</span>
  </div>
  <div class="card">
    <h2>Weight loss until goal</h2>
    <span class="big">${weightLossUntilGoal.toFixed(2)}kg</span>
  </div>
</div>