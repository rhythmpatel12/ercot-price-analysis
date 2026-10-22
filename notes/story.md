# Story: findings log

Findings collected step by step. Each one is 1–3 sentences backed by a number.

## Step 2: Explore by hand (2024 Day-Ahead prices)

**The data.** ERCOT's 2024 Day-Ahead hub and load-zone file (NP4-180-ER) has one sheet per month and 131,760 rows. Each row is one settlement point × one hour. There are 15 settlement points: 7 hubs (`HB_BUSAVG`, `HB_HOUSTON`, `HB_HUBAVG`, `HB_NORTH`, `HB_PAN`, `HB_SOUTH`, `HB_WEST`) and 8 load zones (`LZ_AEN`, `LZ_CPS`, `LZ_HOUSTON`, `LZ_LCRA`, `LZ_NORTH`, `LZ_RAYBN`, `LZ_SOUTH`, `LZ_WEST`). The data is complete: every point has exactly 8,784 rows (366 days × 24 hours), with no missing values.

**Time follows the clock, not the sun.** ERCOT records time in local Texas time, so daylight saving shows up in the data. On March 10, 2024, Hour Ending 03:00 is missing (a 23-hour day). On November 3, 2024, Hour Ending 02:00 appears twice (a 25-hour day), and the second copy has `Repeated Hour Flag = "Y"`. The two copies have different prices (HB_NORTH: $10.49 vs $13.60), so they are two real hours, not a duplicate to drop. Times will need converting to UTC before Day-Ahead and Real-Time prices can be lined up in Step 3.

**Dates and times are text.** `Delivery Date` (`"01/01/2024"`) and `Hour Ending` (`"01:00"` to `"24:00"`) load as strings. `"24:00"` isn't a valid time, so they can't be parsed directly into timestamps.

**Prices:** 
Average: $28.903/MWh, 
Max: $2347.76/MWh (1/16/2024 @ LZ_LCRA) - Price during the texas winter freeze!!, 
Cheapest Hub: HB_PAN with avg- $21.16/MWh
