# Nutrition service: product requirements

Status: Product scope agreed for an owner-only proof of concept. The `Emberpath-nutrition-service` repository exists as a FastAPI template, but nutrition behavior is not implemented or included in Docker Compose. Calculation details, the API contract and release safeguards still need specification during implementation.

## Purpose and language

The first release is a self-service calculator for adults age 19 or older training for strength and fitness, including powerlifting, weightlifting, strongman and streetlifting/calisthenics. It helps them estimate energy needs, choose a daily calorie and nutrient target, and optionally save the accepted result as a nutrition plan. Age eligibility is not the same thing as the intended training audience. Estimates are not measurements, medical advice, or a food-intake record.

- **Estimate:** a provisional calculator result. It does not become a plan unless the user explicitly saves it.
- **Planned daily target:** calories and protein, carbohydrate, fat and fibre intended for a day. It is not a total consumed or burned that day. A sum over selected days, if shown, is a projection of planned targets, not logged intake.
- **Planned weekly calorie total:** seven times the user's chosen average daily calorie target. Changing the allocation across weekdays does not change this total; a new overall target requires an explicit plan replacement.
- **Nutrition plan:** the user's saved choice of inputs, calculation method and version, accepted targets, and overrides, including any weekday allocation. It is distinct from a weight goal, which remains owned by the weight service.

## First-release scope

1. Accept calculation inputs only for users age 19 or older; do not apply an adult equation to younger users. Use Mifflin-St Jeor as the first method for estimating **resting** energy expenditure from manually entered weight, height and age. Ask explicitly for the method's male/female formula parameter, not account identity, and never infer it from Clerk. Allow a manual target when the method is unsuitable for an otherwise eligible user. Explain individual uncertainty; resting energy alone is not estimated maintenance expenditure. Input bounds and rounding remain open.
2. Estimate maintenance separately using the applicable adult total-energy expenditure equation from the [2023 Dietary Reference Intakes for Energy](https://www.nationalacademies.org/read/26818/chapter/2) and an explicit general activity category. Do not present this as a multiplier prescribed by Mifflin-St Jeor. Keep the resting and maintenance estimates, activity assumption and method versions visible and distinct. A general activity category does not model a specific strength sport or training workload; the maintenance estimate remains provisional.
3. Show the user's chosen calorie adjustment and final target separately from either estimate. Support manual overrides before saving. In v1.0 the user enters any calorie adjustment; there are no one-click numeric deficit or surplus presets. Do not call an adjustment universally safe or promise a rate of weight change. Use one editable strategy across the week: protein and fibre targets remain steady each day, fat comes from an editable share of that day's planned calories, and carbohydrate accounts for the remaining energy. Show source-labeled, editable starting suggestions as specified below. The protein starting point reflects this service's training-focused audience; do not infer a sport or change protein automatically from the general activity category. Do not offer independent macro edits for each weekday in v1.0. Reject negative or infeasible nutrient values instead of silently changing the user's choices. Validation bounds and rounding remain open.
4. Let the user save an estimate as a nutrition plan, or leave without saving. Each user has at most one active plan; saving a new version retains the previous one. Capture the accepted inputs, method/version, targets and overrides so later measurement or weight-goal changes never silently rewrite a plan. Recalculation requires an explicit user action. A new plan starts with the same planned target on every day; the user may instead assign different targets to the seven days of the week. The weekday pattern repeats; it does not change the calculator's energy estimate or the chosen planned weekly calorie total. Show the remaining allocation while editing and do not save an unbalanced week. Changing the weekly total instead requires an explicit new overall target and plan version. Saving activates a plan immediately and replaces any active version while retaining it as history. Let the user explicitly end a plan without deleting its history; afterwards no plan is active. Future-dated starts and backdating are not supported in v1.0.
5. Keep plan reads and writes scoped to the authenticated user. The service owns nutrition calculations and plans. The web app owns input and result presentation; the weight service keeps ownership of measurements and weight goals.

## First-run nutrient suggestions

These are starting points for the intended training audience to review and edit before saving, not measured needs, a meal plan or an assessment of individual safety. Fat and fibre references are general-adult guidance, not sport-specific methods. Show each source and its scope beside the suggestion. Save the user's chosen values, not just the defaults, in the plan snapshot.

| Target | Editable starting suggestion | Source and limitation |
| --- | --- | --- |
| Protein | `1.6 g/kg/day` from the manually entered weight, held steady across weekdays. Let the user edit the multiplier or resulting target, including a choice of `2.0 g/kg/day`. | The [ISSN position stand](https://pubmed.ncbi.nlm.nih.gov/28642676/) describes `1.4-2.0 g/kg/day` as sufficient for most exercising individuals. A [resistance-training meta-analysis](https://pubmed.ncbi.nlm.nih.gov/28698222/) found a group-level plateau around `1.6 g/kg/day` for gains in fat-free mass. Neither establishes `1.5 g/kg/day` as a universal minimum or `1.6` as right for every athlete, goal or energy deficit. |
| Fat | `30%` of each day's chosen calorie target, converted using approximately `9 kcal/g`; grams can differ by weekday. | [WHO's general-adult guidance](https://www.who.int/news/item/17-07-2023-who-updates-guidelines-on-fats-and-carbohydrates) discusses a `30%` total-fat limit and the importance of fat quality. Using `30%` as an editable allocation is a product choice, not a prescribed individual target; the calculator does not measure fat quality. |
| Fibre | `25 g/day`, held steady across weekdays. | [WHO's general-adult guidance](https://www.who.int/news/item/17-07-2023-who-updates-guidelines-on-fats-and-carbohydrates) concerns naturally occurring dietary fibre. A numeric plan cannot establish its food sources or actual intake. Do not scale the suggestion down for a lower-calorie weekday. |

Derive carbohydrates from each day's remaining energy after the chosen protein and fat allocations, using approximate `4/9/4 kcal/g` factors. Fibre is included in total carbohydrates, not an additional energy line. Do not save a day whose planned calories cannot accommodate the chosen protein, fat and fibre values under this model; show the conflict for the user to resolve instead of altering another day or the weekly total.

## Release posture

The first deployment is an owner-only, self-service proof of concept. Restrict actual access to the intended account at the service or another verified deployment boundary; hiding the UI is not an access control. Still implement authentication and per-user plan isolation so support for additional users does not require rewriting ownership. Do not present the private PoC as a publicly suitable health product. A wider self-service release needs a separate readiness decision, including review of formula applicability, unsupported health circumstances, day-level targets and safety-related copy. Mathematical feasibility and a balanced week do not establish individual appropriateness.

## Plan lifecycle and calendar

A saved plan becomes active at save time. Saving a replacement retires the earlier active version at that time without rewriting its historical targets. Ending a plan is an explicit action that leaves no active plan and retains the ended version for history. An ended plan is not silently resumed; a later plan requires another explicit save. There are no scheduled activations or retroactive edits in v1.0.

The repeating seven-day pattern is keyed to the user's local weekdays, not to seven days elapsed since saving. The full weekly calorie invariant applies to each complete pattern; a period containing only some active days projects only those days' planned targets. Define and retain the plan's calendar time zone so projections do not depend on the service host's time zone.

## Deferred capabilities

- Food-intake, activity and calorie-burn logging, and actual daily totals.
- Scheduled target curves, such as a desired weight change over 12 weeks, and automated plan changes based on new measurements or weight goals.
- Future-dated or backdated plan activation and retroactive target changes.
- Sport-specific calculation presets, such as powerlifting, and step-based activity presets, until their inputs and effects can be stated and tested. The intended audience does not make a training category an automatic input.
- An optional average step-count input or an authorized wearable source, such as Oura. Neither is assumed to improve the estimate without a validated way to use it and avoid double-counting other activity inputs.
- Automatic weight lookup for the calculator. A later optional web control may copy `GET /weight-logs/summary`'s `latest` measurement into an editable input when recent enough (approximately 14 calendar days). Manual entry must always work without a weight log. That control would be a convenience, not verification by the nutrition service of the measurement's provenance.
- Hydration estimates, targets and tracking; water is not part of v1.0.
- Calculated estimates for people younger than 19; age 18 requires a separately selected method instead of reusing the adult equation.
- A weekly nutrient what-if planner where users independently propose a week's daily protein, carbohydrate, fat and fibre targets and compare their projected totals with the calculated plan. This is the next planned slice after the MVP (provisionally v1.1), not food-intake logging or a second deployed service.

## Acceptance scenarios

- A user age 19 or older can view an estimate without creating a plan; no food or activity log is needed. A younger user receives an explicit unsupported-age response, not an adult-formula result.
- Saving records exactly the accepted result and its assumptions. Changing weight, goals or calculation inputs later leaves the saved version unchanged until the user explicitly saves a replacement.
- A plan starts with flat daily targets but can save an explicit repeating seven-day allocation that sums to the same planned weekly calorie total. Different weekday targets are planned values, not food or activity logs.
- A change to a weekday target cannot silently change another day or the planned weekly total. An unbalanced week cannot be saved.
- The plan uses one editable nutrient strategy across the week. Protein and fibre remain steady while fat and carbohydrate targets reflect each day's saved calorie allocation. The first-run suggestions show their sources and can be changed before saving; saved choices remain unchanged when future defaults change. Invalid or infeasible overrides, including more fibre than total carbohydrate on a day, produce an explicit error, not an adjusted success result.
- With `80 kg` entered manually, the protein starting calculation is `128 g/day` before rounding (`1.6 g/kg/day`); the user can instead choose `2.0 g/kg/day` (`160 g/day`). Changing the energy activity category alone does not change the chosen protein multiplier or a saved plan.
- Saving activates the chosen plan immediately. Replacing it retains the previous version, and ending it leaves no active plan without deleting either version. A partial first week projects only its active calendar days; the repeating full-week targets still preserve the weekly total.
- Another authenticated user cannot read or replace this user's plans.
- In the private deployment, an otherwise authenticated account not admitted as the owner cannot use the nutrition API merely by accessing its route.
- A user without weight measurements can complete the first-release flow.
- The UI distinguishes estimated expenditure, chosen targets, a saved plan and any future actual intake; it does not present a projected target as consumed.

## Implementation sequence and ownership

1. **Contract and safeguards (backend lead; web reviews):** replace template metadata and define a typed, versioned estimate/plan API with units, method identifiers, errors and calendar semantics before client wiring. Keep the existing weight API unchanged. Route browser-facing nutrition requests under `/api/nutrition/v1/...` through the web proxy to the service's `/api/v1/...`. Protect domain endpoints with verified Clerk identity; restrict the private deployment to the owner's verified issuer and subject, not a hidden UI or email address. Define the numeric limits, rounding and day-level risk handling before enabling plan saves.
2. **Calculator (backend calculation module, then web preview):** implement and reference-test versioned resting and maintenance equations, the nutrient strategy, adjustments and explicit invalid-input results. Expose an authenticated estimate that creates no plan. The web owner builds manual inputs, an editable source-labeled preview and clear error states against the agreed contract; no weight-service lookup is required.
3. **Plans (backend plan module, then web plan flow):** use nutrition-owned storage and migrations, never weight-service tables. Atomically save, replace and end immutable plan versions per user. Enforce the weekly calorie invariant, each day's nutrient feasibility and the calendar time-zone contract, including partial weeks and concurrent saves. The web owner adds the repeating weekday editor, remaining-allocation display, history and explicit save/replace/end actions.
4. **Integration (web proxy and hub Compose):** route nutrition separately from the existing weight API in Vite and Nginx, then configure a nutrition-owned database, migration step, service readiness and access settings in the hub. Verify owner-only denial, user isolation, the calculator-to-plan flow and unaffected weight behavior. Obtain an independent implementation review and a security review of the new authentication and access boundary before private deployment. A wider release requires the separate readiness decision described above.

## Open decisions and risks

- Specify the published Mifflin-St Jeor equation, rounding, input bounds, limitations, and handling of people for whom its assumptions do not apply. The original [study](https://pubmed.ncbi.nlm.nih.gov/2305711/) and a comparative [review](https://pubmed.ncbi.nlm.nih.gov/15883556/) inform this choice; neither establishes an individual's actual expenditure.
- Validate the applicable 2023 adult maintenance equation, activity-category definitions, input units, rounding and reference cases. Define adjustment input bounds, how to handle unsupported circumstances for an eligible adult, and which other values a user may override.
- Verify nutrient-method applicability and define exact rounding and day-level validation, including conflicting overrides, against reference cases. A balanced weekly calorie total alone does not establish that each individual day's target is appropriate. Specify the plan calendar time-zone contract and date-boundary reference cases, including partial weeks.
- Decide how to handle unsupported health circumstances without implying that an automatically suggested target is suitable for every adult. Complete safety and copy review before any wider self-service release.
- Complete the protected API, nutrition-owned persistence and independent service configuration in the implementation sequence above. A future verified weight reference would require an explicit cross-service authorization contract; v1.0 relies on manual weight input.

The suggested calculation shortcuts and their evidence gaps are assessed in [calculation notes](calculation-evidence.md). Similar numerical reference values do not make those shortcut formulas the selected methods.

One deployable nutrition service initially owns separate calculation and plan modules. This is not a commitment to combine future food or activity logging with it. The chosen boundaries are recorded in [ADR 0001](../adr/0001-nutrition-calculation-and-plan-boundaries.md) and [ADR 0002](../adr/0002-nutrition-plan-snapshots.md). The weekly allocation invariant is recorded in [ADR 0003](../adr/0003-preserve-planned-weekly-calories.md).
