### How to zero-in on the Tier-5 / Lien-Title problem

*(Comprehensive plan – written in plain, “just-tell-me-what-to-do” language)*

---

## 1. Why Tier-5 deserves the spotlight (30-second refresher)

| Tier path   | Days past due | Time spent in tier | What happens                              |
| ----------- | ------------- | ------------------ | ----------------------------------------- |
| **Tier 1**  | 75-DPD start  | 22 days            | Early collection calls                    |
| **Tier 2**  | next 60 days  | 60 days            | Escalated outreach                        |
| **Tier 3**  | next 60 days  | 60 days            | Legal letters begin                       |
| **Tier 4**  | next 60 days  | 60 days            | Repossession prep                         |
| **Tier 5**  | next 60 days  | 60 days            | Collateral assigned, title work triggered |
| **Dormant** | +90 days wait | 90 days            | Loan “rests,” then cycles back to Tier 1  |

**Why we care:**
Lien/Title errors almost always surface **while a loan sits in Tier 5** (title search & repo paperwork). The longer or more often a loan stays there, the higher the risk.

---

## 2. Data you need (quick checklist)

1. **Loan & repo facts**

   * Loan ID, Origination date, State / county
   * Repo date, Repo amount (\$)
   * RIE flag (Y/N) and RIE type
2. **Tier timeline** – one row per tier change

   * Loan ID, Tier number, Tier entry date, Tier exit date
   * You can derive if raw dates aren’t logged (use DPD + your “22/60/60” rules).
3. **QC timestamps** (if available)

   * Title request date, Title received date, QC approval date.

---

## 3. Simple analysis game-plan (step by step)

### Step A – Build “days-in-Tier-5” features

```text
days_in_t5_before_repo   =  date_repo – tier5_entry_date
cycles_in_t5             =  floor(days_in_t5_before_repo / 60)
```

### Step B – Bucket loans for an apples-to-apples view

| Bucket name     | Logic                             | Example                               |
| --------------- | --------------------------------- | ------------------------------------- |
| **No T5**       | Loan never reached Tier 5         | cycles\_in\_t5 = 0 & days\_in\_t5 = 0 |
| **Early T5**    | In Tier 5 < 30 days               | 0 < days\_in\_t5 ≤ 30                 |
| **Full Cycle**  | 30 < days ≤ 60                    |                                       |
| **Multi-Cycle** | days > 60 *or* cycles\_in\_t5 ≥ 1 |                                       |

### Step C – Measure the hit rate

For each bucket calculate:

```text
LienTitle_RIE_%  =  (# Lien/Title RIE)  ÷  (# repos in bucket)
```

Plot as a simple **clustered bar**: buckets on X, error % on Y.

### Step D – Quick statistical test

* Run a **chi-square test** on the 4 × 2 table (bucket vs. RIE yes/no).
* If p < 0.05 and the “Multi-Cycle” bucket has 2-3× the error rate, Tier-5 exposure is officially a culprit.

*(No fancy ML needed—though a logistic regression with `days_in_t5` as a predictor will give the same answer.)*

---

## 4. Turning insight into action (“pilot first, scale later”)

| Pain-point uncovered                       | 2-week pilot fix                                                              | Metric to watch                                  |
| ------------------------------------------ | ----------------------------------------------------------------------------- | ------------------------------------------------ |
| Multi-Cycle loans show 3× more L/T errors  | **Auto-escalate at day-45 of Tier 5**: senior analyst re-reviews title packet | L/T RIE % for >60-day loans should fall ≥ 0.5 pp |
| Early T5 loans (0-30 days) have few errors | **Fast-track repos**: skip dormant 90-day wait if all QC green                | Days-to-repo drops without L/T spike             |
| QC timestamps show wide variance           | **Trigger reminder email** at 10 days if title doc not yet in                 | Median “request→receipt” time shrinks            |

---

## 5. Keeping it simple to monitor

1. **Weekly Tier-5 dashboard**:

   * Bar chart: L/T RIE % by bucket (No T5, Early, Full, Multi)
   * Line chart: rolling 4-week L/T RIE % overall
2. **Traffic-light KPI** on your Yammer/Teams channel:

   * Green ≤ 25 %, Yellow 25–30 %, Red > 30 %.

---

## 6. Timeline recap (at-a-glance)

| Week    | Deliverable                                                         |
| ------- | ------------------------------------------------------------------- |
| **1**   | Pull raw data; replicate mentor’s 3 key visuals for sign-off        |
| **2**   | Build Tier timeline; produce bucketed error-rate bar chart          |
| **3**   | Present findings; agree on pilot scope (e.g., day-45 senior review) |
| **4–5** | Run pilot; track weekly dashboard                                   |
| **6**   | Decide: scale, tweak, or scrap the pilot; document SOP changes      |

---

### End-result

You’ll be able to walk into your next check-in and say:

> “We proved loans sitting in Tier 5 > 60 days carry a **31 %** Lien/Title error rate—**2.8×** the normal rate. After we added a day-45 senior QC last week, that bucket’s error rate fell to **24 %**. Let’s roll the QC step out to all Tier-5 loans immediately and revisit in 30 days.”

That’s the kind of story—and action plan—your mentor is after.
