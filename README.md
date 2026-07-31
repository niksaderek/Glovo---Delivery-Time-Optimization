# Glovo — Delivery Time Optimization

### Everyone blamed distance. Distance had an alibi.

---

A week of food delivery in the city of Glovalia: 2,471 orders, 83 couriers, 68 restaurants. Somewhere in that week, roughly 270 dinners showed up late enough that the customer noticed.

The number that matters to the business is simple. What share of food orders reach the customer within 45 minutes? Glovo wants 85%. That week it got **73.47%** — eleven and a half points short, which in a single city is a few hundred disappointed people every seven days.

So: where does an order lose its eleven minutes?

![Delivery time by hour of the day](docs/images/01-hero-delivery-time-by-hour.png)

Look at the shape of that chart before reading on. Delivery times sit flat and healthy through the whole working day, and then, somewhere around eight in the evening, they come apart. That's the first hint that whatever is going wrong isn't a constant — it's something that only breaks under load.

---

## The suspect everyone likes

Ask anyone why food delivery runs late and they'll tell you the couriers are travelling too far. It's the intuitive answer, and it comes with an expensive, obvious fix: cap the delivery radius.

The data doesn't back it. Across all orders, the correlation between distance travelled and time taken is **0.32** — real, but weak. Distance explains roughly a tenth of what's going on.

Then comes the part that settles it. Isolate only the orders that actually breached 45 minutes — the failures, the ones we're trying to explain — and the correlation *drops* to **0.23**.

Distance matters least precisely where it would need to matter most. Whatever makes an order late, it isn't the length of the trip. It's everything that happens before the courier ever starts moving.

---

## Suspect one: the handoff

Every order gets assigned to a courier. Sometimes that courier doesn't take it, and the order gets passed to another. It sounds like a minor administrative event. It is not.

![Delivery time by number of courier assignments](docs/images/02-delivery-time-by-assignments.png)

The median line crosses the target the instant a second courier gets involved.

| Assignments | Average time | Share of orders |
|---|---|---|
| 1 | **36 min** — comfortably inside | 86.9% |
| 2 | **47 min** — outside | 10.7% |
| 3 | 55 min | 2.1% |
| 4 | 80 min | 0.3% |

One handoff costs 23.6% of the delivery time. That single step is the difference between a customer who's happy and a customer who's watching the app.

Most orders never touch it — 87% are matched once and go. But the ones that do reassign do real damage: **318 orders, 13.3% of the entire week, missed the target with a reassignment attached to them.**

There's a darker version of the same story. Orders that were eventually cancelled had been passed around an average of 1.46 times, against 1.17 for orders in general. Reassignment isn't only a delay — past a certain point it's how an order dies.

---

## Suspect two: the eleven minutes nobody counts

If the trip isn't the problem, the problem has to be somewhere in the choreography before it. So break a delivery into its six real stages and time each one.

![Delivery stage breakdown](docs/images/10-delivery-stage-breakdown.png)

| Stage | Median |
|---|---|
| Order activates → courier assigned | 1.2 min |
| Courier assigned → courier starts | 2.8 min |
| Courier starts → arrives at restaurant | 6.4 min |
| **Standing in the restaurant, waiting** | **11.8 min** |
| Restaurant → customer *(the actual driving)* | 7.6 min |
| Finding the door | 3.3 min |

Read those two bolded rows together, because that's the finding.

**Couriers spend more time waiting for food than they spend delivering it.** Eleven point eight minutes leaning against a counter, versus seven point six actually on the road. The single largest block of time in the whole journey is a courier standing still.

And this is the good news, because that time is free. It doesn't require more couriers, more vehicles, or a smaller delivery radius. It requires the restaurant to start cooking at a moment that bears some relationship to when the courier walks in. Right now those two clocks aren't talking to each other.

---

## Suspect three: a three-hour window

Remember the evening collapse in the opening chart. Here's what's underneath it.

![Orders per hour](docs/images/05-orders-by-hour.png)

Orders peak between 19:00 and 21:00. Delivery times peak, slightly behind them, between 20:00 and 23:00 — the lag you'd expect when a system takes on more than it can absorb and spends the next hour catching up.

![Orders per day](docs/images/06-orders-by-day.png)

The week has the same shape. Wednesday, Saturday and Tuesday run the longest deliveries; Saturday, Wednesday and Sunday carry the most orders.

None of this is random, and that's what makes it useful. The strain arrives on a schedule. Courier supply, right now, does not.

---

## Two loose ends

**The cars.** Motorbikes average 58.6 minutes; cars average 63 and get reassigned noticeably more often (1.45 versus 1.31) — despite covering essentially the same distance. Something about running a car in a dense town costs time that has nothing to do with the driving, and parking is the obvious guess. I'd flag it rather than claim it; this dataset can't see a courier circling for a space.

![Delivery time by transport type](docs/images/08-delivery-time-by-transport.png)

**The cancellations.** 3.32% of orders were cancelled, which is a perfectly respectable number and easy to wave through. But it isn't spread evenly. Restaurant `18300` and restaurant `30640`, between them, account for **39% of every cancellation that week.** That's not a platform problem to be solved with policy. That's two phone calls.

---

## What to do about it

Four weeks, one bottleneck at a time — introduced sequentially, so each one's contribution stays visible rather than tangled up with the others.

| Week | The move | KPI after | Gain |
|---|---|---|---|
| — | Where we started | 73.47% | — |
| 1 | Cut reassignments — better matching, acceptance incentives | 76.43% | +2.96 |
| 2 | Staff the peak, discount the trough | 77.78% | +1.35 |
| 3 | Move car deliveries onto motorbikes | 79.63% | +1.85 |
| 4 | Attack the restaurant wait | 82.97% | +3.34 |
| 5 | Chase the cancellation outliers | 83.23% | +0.26 |
| 6 | Speed up initial assignment | **84.63%** | +1.40 |

Four weeks recovers 9.5 of the missing 11.5 points. Six weeks lands at 84.63% — close enough to touch the target.

The two interventions that pay best, weeks 1 and 4, are the same two bottlenecks the investigation turned up first. That's not a coincidence, it's the whole argument: find where the time actually goes, and the plan writes itself.

> **One caveat, stated up front.** The week-by-week figures come from a simulation over 10,000 synthetic deliveries, and the improvement coefficients in it were *chosen, not causally estimated from the data.* What the simulation demonstrates is the **ordering and rough magnitude** of the levers — that fixing reassignments and restaurant waiting beats fixing distance. It is not a forecast, and real effect sizes need in-market A/B testing to establish. I'd rather say that plainly here than let a reader mistake a model for a promise.

---

## Run it yourself

```bash
git clone https://github.com/niksaderek/Glovo---Delivery-Time-Optimization.git
cd Glovo---Delivery-Time-Optimization
pip install pandas numpy matplotlib seaborn openpyxl
jupyter notebook notebook.ipynb
```

The data ships with the repo. Run the notebook top to bottom and every number and figure above regenerates from source.

**How it works:** parse seven timestamp fields into a consistent shape → drop cancelled and non-food orders, since neither can be judged against a 45-minute food KPI (2,471 → 2,322) → derive `delivery_time` as termination minus activation → decompose each order into its six stages, which is what makes the waiting problem visible at all → segment by assignment count, hour, weekday, transport, restaurant and courier → isolate outliers per stage with IQR → simulate.

**Built with:** Python · pandas · NumPy · Matplotlib · seaborn · Jupyter

---

*An anonymised operational dataset, analysed as a portfolio case study. Restaurant and courier IDs are pseudonymous integers; there's no personal data here.*
