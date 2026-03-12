# SmartSettle — Payment Routing & Settlement Optimizer

## Algorithm Overview

SmartSettle uses a **deadline-aware greedy scheduling algorithm** to route payment transactions across three settlement channels (FAST, STANDARD, BULK) while minimizing total system cost.

## How It Works

### Step 1: Priority Sorting
Transactions are sorted before scheduling using a two-level comparator:
- **Primary:** Tightest absolute deadline first (`arrival_time + max_delay`)
- **Secondary:** Highest value score (`priority × amount`) to protect high-value transactions when deadlines tie

This ensures urgent transactions get first pick of available channel slots.

### Step 2: Channel Selection
For each transaction, all three channels are evaluated. The algorithm finds the **earliest available slot** on each channel (respecting concurrent capacity limits) and computes the true cost:

```
cost = channel_fee + (P × amount × delay)
where delay = start_time - arrival_time, P = 0.001
```

The channel with the **lowest total cost** is selected, provided the slot is available before the transaction deadline.

### Step 3: Graceful Failure
If no channel can schedule a transaction before its deadline, or if the failure penalty is cheaper than the best available option (for low-priority transactions), the transaction is intentionally marked as failed:

```
failure_penalty = 0.5 × amount
```

This avoids wasting fast channel capacity on low-value transactions.

## Routing Strategy

| Scenario | Channel Chosen |
|---|---|
| Tight deadline (max_delay ≤ 2) | FAST — only channel guaranteed to fit |
| Medium deadline, high amount | STANDARD — balances fee vs delay penalty |
| Large deadline, any amount | BULK — cheapest fee, fits comfortably |
| BULK/STANDARD at capacity | Escalates to next faster channel |

## Complexity

- **Time:** O(n log n) for sorting + O(n × c × k) for scheduling, where n = transactions, c = 3 channels, k = capacity search depth (≤ 2000)
- **Space:** O(n) for slot tracking

## Parameter Choices

- `PENALTY_FACTOR = 0.001` — as specified in problem statement
- `FAILURE_FACTOR = 0.5` — as specified in problem statement
- Sort by absolute deadline (not relative) — ensures globally tightest constraints are resolved first
- Graceful failure only triggers for `priority = 1` transactions where failure is cheaper

## How to Run

```bash
npm install
npm run dev
```

Open `http://localhost:8080`, upload `transactions.csv`, and click **Export submission.json**.
