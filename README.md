# AI-Driven Production Optimization for Mature Oil Fields

A case study on the Equinor Volve field, showing how machine learning can recommend choke (valve) settings that increase oil production while keeping water production under a limit.

## Problem
In a mature oil field, wells produce less oil and more water over time. The platform can only handle a limited amount of water per day. Opening a well's choke can give more oil, but it often brings up much more water.

**Question:** How should each well's choke be set to get the most oil without producing more water than today?

## Data
- Equinor Volve open dataset: daily production data for 5 producing wells (15/9-F-12, F-14, F-11, F-15 D, F-1 C).
- Source and licence: https://www.equinor.com/energy/volve-data-sharing
- The raw data is not included in this repo, in line with the Equinor Open Data Licence. Download it from the link above.

## Method
1. **Cleaning:** kept producer wells only, removed shut-in days and rows with missing values.
2. **Forecast comparison:** compared the classic Arps decline curve with an XGBoost model (inputs: choke size, wellhead pressure, downhole pressure, hours on stream). The first 80% of each well's history was used for training and the last 20% for testing. Error is reported as weighted MAPE, which small end-of-life rates cannot distort.
3. **Surrogate models:** trained an XGBoost model per well for oil rate and one for water rate, as a function of choke size.
4. **Optimization:** tried every combination of 12 choke settings per well and picked the one giving the most total oil while total water stays at or below a limit.
5. **Realistic version:** repeated the optimization with each choke allowed to move at most 15 percentage points from today's setting.

## What is the "water limit"?
The maximum total water the platform can handle per day. In this project it is **assumed**, not Volve's real capacity: the limit is set as a percentage of today's total water rate (about 6,726 Sm3/d). For example, "90% of today" means total water must be cut by 10%.

## Results

**Forecast error on unseen data (weighted MAPE, lower is better)**

| Well | Arps decline | XGBoost |
|---|---|---|
| 15/9-F-12 | 32.1% | 33.2% |
| 15/9-F-14 | 123.9% | 40.6% |
| 15/9-F-11 | 96.5% | 46.4% |
| 15/9-F-15 D | 23.1% | 21.5% |
| 15/9-F-1 C | 39.3% | 41.3% |

XGBoost was clearly better on wells whose operating conditions changed over time (F-14, F-11) and about equal on smoother wells.

**Choke optimization (model predictions, total water kept at today's level)**

| Scenario | Total oil | Change |
|---|---|---|
| Today (model estimate) | 1,019 Sm3/d | - |
| Unrestricted choke moves | 1,230 Sm3/d | +20.7% (upper bound, needs unrealistic choke jumps) |
| Realistic moves (max 15 points) | 1,071 Sm3/d | **+5.2%** |

Main realistic recommendation: ease F-11 from 98.8% to 91.4% (saves about 160 Sm3/d of water for a small oil loss) and open F-1 C from 49.7% to 62.1% (oil rises from 189 to 259 Sm3/d).

Charts and tables are in the `results/` folder.

## Limitations
- The models learn patterns in historical data, not cause and effect. The predicted gains have not been tested in the field.
- Only 5 wells, and test errors of 20-45% mean the predicted gain is uncertain.
- Most of the realistic gain comes from one well (F-1 C).
- Water cuts of 10-20% were not achievable with small choke moves, so large water reductions would need other measures.
- The water limit is an assumption, not Volve's real equipment capacity.

## Next steps
- Add uncertainty ranges to the predictions
- Extend to gas-lift and ESP optimization
- Add injector-producer (waterflood) connectivity
- Build a live dashboard with Streamlit

## Author
KARUNNYA JADHAV (karunnya.jadhav24@gmail.com)
