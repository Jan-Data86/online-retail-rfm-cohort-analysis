## SQL Analysis: Products & Basket

Performed in MySQL Workbench on the cleaned dataset exported from the Python 
pipeline (`online_retail_clean_py`, 1,024,239 rows).

### Setup notes
- Loaded via `LOAD DATA INFILE`, converting empty CustomerID strings to real SQL `NULL`
- Added an index on `Invoice` to support join performance
- Hit `net_read_timeout`/`net_write_timeout` disconnects (default 30s) on the 
  self-join query below — resolved via `SET GLOBAL net_read_timeout = 600; 
  SET GLOBAL net_write_timeout = 600;`

### 1. Top 10 products by revenue

| StockCode | Description | Total Revenue |
|---|---|---|
| 22423 | REGENCY CAKESTAND 3 TIER | 314,045.02 |
| 85123A | WHITE HANGING HEART T-LIGHT HOLDER | 251,603.12 |
| 47566 | PARTY BUNTING | 147,079.73 |
| 85099B | JUMBO BAG RED RETROSPOT | 145,946.05 |
| 84879 | ASSORTED COLOUR BIRD ORNAMENT | 128,550.42 |
| 22086 | PAPER CHAIN KIT 50'S CHRISTMAS | 116,298.59 |
| 79321 | CHILLI LIGHTS | 79,905.13 |
| 84347 | ROTATING SILVER ANGELS T-LIGHT HLDR | 70,969.95 |
| 85099F | JUMBO BAG STRAWBERRY | 68,489.89 |
| 23084 | RABBIT NIGHT LIGHT | 66,661.63 |

**Insight:** decorative/gift items dominate the top of the revenue ranking — 
consistent with this being a UK-based gift and decor retailer.

### 2. Market basket analysis (top 10 co-purchased product pairs)

| Product A | Product B | Times bought together |
|---|---|---|
| 22386 | 85099B | 1,492 |
| 21931 | 85099B | 1,378 |
| 21733 | 85123A | 1,340 |
| 20725 | 20727 | 1,273 |
| 21212 | 84991 | 1,262 |
| 85099B | 85099F | 1,246 |
| 20725 | 22383 | 1,225 |
| 20725 | 22384 | 1,218 |
| 21231 | 21232 | 1,204 |
| 22411 | 85099B | 1,199 |

**Insight:** `85099B` (JUMBO BAG RED RETROSPOT, also #4 by revenue) appears in 
four of the top 10 pairs. This likely reflects its high standalone popularity 
rather than a unique product affinity — raw co-occurrence count doesn't 
control for how often each item appears on its own. A Lift-based metric 
(comparing observed vs. expected co-occurrence under independence) would be 
the natural next step to distinguish true affinity from base popularity.

### 3. Top 10 products by cancellation rate

| StockCode | Total Orders | Cancels | Cancel Rate |
|---|---|---|---|
| 79323B | 109 | 51 | 46.8% |
| 79323GR | 120 | 37 | 30.8% |
| 84770 | 23 | 7 | 30.4% |
| 23064 | 33 | 10 | 30.3% |
| 79323G | 87 | 25 | 28.7% |
| 84816 | 28 | 8 | 28.6% |
| 79323S | 78 | 22 | 28.2% |
| 79323W | 476 | 126 | 26.5% |
| 79323LP | 230 | 60 | 26.1% |
| 79323P | 352 | 86 | 24.4% |

**Insight:** 8 of the top 10 products belong to a single line — CHERRY LIGHTS 
(StockCode prefix `79323`, one code per color variant). Investigating further 
(query 7) revealed inconsistent `Description` values for the same StockCode 
across rows — duplicates with different spacing and even empty descriptions 
for some codes (e.g. `79323G`, `79323W`). This suggests catalog data 
inconsistency as a plausible contributing factor to the elevated cancellation 
rate, rather than necessarily a product defect.