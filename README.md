# Glovo — Delivery Time Optimization

**Everyone blamed distance. Distance wasn't the problem.**

One week of Glovo food delivery in Glovalia: 2,471 orders, 83 couriers, 68 stores. Only **73%** arrived within 45 minutes. Target: **85%**.

That's ~270 late orders a week in one city. Here's where they went.

![Delivery time by hour of the day](docs/images/01-hero-delivery-time-by-hour.png)

---

## The wrong suspect

Distance-to-delivery-time correlation: **r = 0.32**. Among orders that actually blew the deadline: **0.23**.

It gets *weaker* on the late orders. Capping courier travel would have been an expensive fix aimed at the wrong variable.

The real culprit is coordination — matching, handoffs, and standing around in restaurants.

---

## Suspect 1: the second courier

Hand an order to a new courier and the clock resets.

![Delivery time by number of courier assignments](docs/images/02-delivery-time-by-assignments.png)

| Assignments | Avg time | Orders |
|---|---|---|
| 1 | **36 min** ✅ | 86.9% |
| 2 | **47 min** ❌ | 10.7% |
| 3 | 55 min | 2.1% |
| 4 | 80 min | 0.3% |

One reassignment costs **+23.6%** and pushes the median straight through the target line. **318 orders — 13.3% of the week — were late with a reassignment attached.**

Reassignments predict cancellations too: cancelled orders averaged 1.46 assignments vs 1.17 overall.

---

## Suspect 2: 11.8 minutes of standing still

Split the journey into six stages and the delay isn't on the road:

![Delivery stage breakdown](docs/images/10-delivery-stage-breakdown.png)

| Stage | Median |
|---|---|
| Activation → assignment | 1.2 min |
| Activation → courier start | 2.8 min |
| Activation → pickup arrival | 6.4 min |
| **⏳ Waiting at pickup** | **11.8 min** |
| Store → customer *(actual driving)* | 7.6 min |
| Time in delivery zone | 3.3 min |

**Couriers spend more time waiting for food than delivering it.** That's store prep-time misaligned with courier arrival — fixable without hiring one extra courier.

---

## Suspect 3: a three-hour window

![Orders per hour](docs/images/05-orders-by-hour.png)

Slowest deliveries: **20:00–23:00**. Peak volume: **19:00–21:00**. Worst days: **Wednesday, Saturday, Tuesday**.

![Orders per day](docs/images/06-orders-by-day.png)

The strain isn't spread across the week. It's concentrated — which means courier supply should be too.

---

## Two smaller ones

**Cars are the slowest vehicle** (63 min avg, 1.45 assignments) despite covering no more ground than motorbikes (58.6 min, 7.2 km). Parking friction, most likely — the data can't confirm the mechanism.

![Delivery time by transport type](docs/images/08-delivery-time-by-transport.png)

**Cancellations have names.** 3.32% overall — fine. But store `18300` plus store `30640` account for **39% of them**. Two conversations, not a systemic fix.

---

## The plan, simulated

Each week attacks one bottleneck, introduced sequentially so the contribution stays measurable.

| Week | Move | KPI | Δ |
|---|---|---|---|
| — | Baseline | 73.47% | — |
| 1 | Cut reassignments | 76.43% | +2.96 |
| 2 | Peak-hour slots + off-peak pricing | 77.78% | +1.35 |
| 3 | Cars → motorbikes | 79.63% | +1.85 |
| 4 | Kill pickup waiting | 82.97% | +3.34 |
| 5 | Lower cancellations | 83.23% | +0.26 |
| 6 | Faster assignment | **84.63%** | +1.40 |

**Four weeks closes 9.5 points. Six lands at 84.63% — 0.4 short of target.**

The two biggest payoffs are the two bottlenecks the analysis flagged first. That's the point.

> ⚠️ **The simulation is illustrative, not a forecast.** It runs 10,000 synthetic deliveries with improvement coefficients that are *chosen, not causally estimated*. It shows the ordering and rough magnitude of the levers. Real effect sizes need in-market A/B testing. Flagging this loudly because anyone acting on these numbers deserves to know which half is measured and which is modelled.

---

## Run it

```bash
git clone https://github.com/niksaderek/Glovo---Delivery-Time-Optimization.git
cd Glovo---Delivery-Time-Optimization
pip install pandas numpy matplotlib seaborn openpyxl
jupyter notebook notebook.ipynb
```

Data's included. Everything regenerates from source — cleaning, KPI, every figure.

**Method:** parse 7 timestamp fields → drop cancelled + non-food (2,471 → 2,322 orders) → `delivery_time = termination − activation` → decompose into 6 stages → segment by assignment count, hour, weekday, transport, store, courier → IQR outliers per stage → simulate.

**Stack:** Python · pandas · NumPy · Matplotlib · seaborn · Jupyter

---

*Anonymised operational dataset, portfolio case study. Store and courier IDs are pseudonymous integers — no personal data.*
