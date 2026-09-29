# Calculation shortcuts: evidence and open choices

Status: Discovery note, not a formula specification or individualized advice.
The proposed rules below assume body weight in kilograms and energy in
kilocalories per day; they are not interchangeable estimates of expenditure,
nutrient needs and hydration.

| Suggested rule | What it actually does | First-release decision |
| --- | --- | --- |
| `weight * 26` for maintenance kcal | A weight-only shortcut; it omits height, age, the explicit formula parameter and activity. It is not established here as a general maintenance estimator. | Do not use as the default. Mifflin-St Jeor estimates **resting** energy; use a separately named adult energy-requirement equation for estimated maintenance. |
| `weight * 2` for daily protein grams | Chooses 2 g/kg/day, toward the upper end of the [ISSN range for most exercising individuals](https://pubmed.ncbi.nlm.nih.gov/28642676/). It is a legitimate user choice, not a universal minimum or a requirement for every strength athlete. | Offer an editable `1.6 g/kg/day` training-focused starting point; show the evidence-supported `1.4-2.0 g/kg/day` context and permit `2.0`. A [resistance-training meta-analysis](https://pubmed.ncbi.nlm.nih.gov/28698222/) found a group-level plateau near `1.6` for fat-free-mass gains, not a hard ceiling for individuals or for other goals. |
| `maintenance kcal / 30` for daily fat grams | At approximately 9 kcal/g of fat, assigns 30% of those calories to fat. That is an allocation choice, not a fat requirement; [WHO's 2023 guidance](https://www.who.int/news/item/17-07-2023-who-updates-guidelines-on-fats-and-carbohydrates) discusses a 30% total-fat limit and fat quality. | Offer an editable 30% starting share based on **each chosen daily target**, not maintenance kcal. This calculation does not assess fat quality. |
| `maintenance kcal * 0.014` for daily fibre grams | Equals 14 g per 1,000 kcal, a population-level fibre reference in the [Dietary Reference Intakes](https://www.nationalacademies.org/read/10490/chapter/2), not a personalized prescription. | Instead offer an editable, steady 25 g/day starting value from [WHO's general-adult naturally occurring fibre guidance](https://www.who.int/news/item/17-07-2023-who-updates-guidelines-on-fats-and-carbohydrates). Do not lower it automatically for a lower-calorie weekday. |
| `(kcal - protein_g * 4 - fat_g * 9) / 4` for daily carbohydrate grams | Balances the remaining energy as carbohydrate under approximate 4/9/4 kcal-per-gram factors. It does not choose appropriate protein or fat targets, and it needs explicit treatment of fibre, rounding and overrides. | Balance against the **chosen daily** calorie target, not maintenance; fibre is part of total carbohydrates. Reject days whose carbohydrate allocation cannot accommodate the chosen fibre. |
| `weight / 30` for daily water litres | Interpreted with kg and litres, this is about 33 ml/kg/day. [Total-water guidance](https://www.nationalacademies.org/read/10925/chapter/2) includes food and other beverages, and needs vary with conditions and activity. | Defer hydration beyond v1.0; do not adopt this as a universal requirement. |
| `-500` or `+500` kcal as a quick adjustment | A number added to or subtracted from estimated maintenance; it does not establish safety or predict a fixed rate of weight change. The cited [weight-management guideline](https://pubmed.ncbi.nlm.nih.gov/24222017/) addresses a specific clinical population, not a universal self-service preset. | V1.0 uses user-entered adjustments only; neither number is a one-click preset or labeled safe. |

The selected first resting-energy equation is
[Mifflin-St Jeor](https://pubmed.ncbi.nlm.nih.gov/2305711/), supported as a
general adult starting point by a [comparative review](https://pubmed.ncbi.nlm.nih.gov/15883556/).
For estimated maintenance, the [2023 Dietary Reference Intakes for Energy](https://www.nationalacademies.org/read/26818/chapter/2)
provide separate total-energy-expenditure equations for men and women age 19
and older by activity category. They are the selected v1.0 maintenance
method, subject to verifying input eligibility and calculation reference
cases. Do not silently treat a generic multiplier of resting energy as a
validated individual measurement. Resting and maintenance values use
different methods and both remain estimates. A general activity category
does not make the maintenance method strength-sport specific.

The first-run nutrient values in the [PRD](prd.md) are editable starting
references, not direct measurements or guaranteed safe individual targets.
Review question (2026-09-29): what protein suggestion suits a
strength-training-focused adult calculator, and what fat and fibre references
inform the accompanying provisional targets? The source check covered the
[ISSN position stand](https://pubmed.ncbi.nlm.nih.gov/28642676/) (2017,
healthy exercising individuals, `1.4-2.0 g/kg/day` sufficient for most),
the [Morton and colleagues resistance-training systematic review and
meta-analysis](https://pubmed.ncbi.nlm.nih.gov/28698222/) (2018, group-level
fat-free-mass gains plateau near `1.6 g/kg/day`), [EFSA's general-adult
population reference](https://www.efsa.europa.eu/en/press/news/120209)
(2012, `0.83 g/kg/day`, *not* the chosen training-focused suggestion), and
[WHO's adult fat and fibre guidance](https://www.who.int/news/item/17-07-2023-who-updates-guidelines-on-fats-and-carbohydrates)
(2023). It excludes clinical diets, pregnancy, age under 19, weight-loss-rate
claims and hydration. The position stand notes that some resistance-trained
people in an energy deficit may need more protein; neither source establishes
a universally safe minimum or maximum for every strength sport.
The `1.6 g/kg/day` protein, `30%` fat share, steady `25 g/day` fibre and
approximately `4/9/4` carbohydrate balancing are product starting choices,
not an individually validated nutrition method. Numeric input bounds,
rounding and day-level safety review remain open.
