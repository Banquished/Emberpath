# Next service: calorie calculator and diet planning

Status: ideas to scope after weight import/export is complete; not implemented.

## Intended capabilities

- Calculate calorie targets for maintenance, deficit and surplus.
- Show daily targets and totals across a chosen period.
- Include protein, carbohydrate, fat and fibre targets.
- Later offer editable activity/training presets, such as powerlifting or a daily
  step-count scenario.

## Decisions to make before implementation

- Define the initial inputs and select/document the calculation methods.
- Distinguish estimated expenditure, user-selected adjustments and saved plans.
- Decide whether macro targets use percentages, grams per body mass, or explicit
  user values, and how inconsistent inputs are handled.
- Define what each activity preset actually changes; a label alone is not a
  reliable activity estimate.
- Decide how a saved plan responds to a changed weight or weight goal: retain its
  original inputs, offer recalculation, or create a new plan version.

Weight-service remains the owner of measurements and weight goals. The new
service should read those through an authorized API and own its own nutrition
calculations and plans. Unit/display preferences remain deferred to future user
preferences. Naming, repository creation and implementation scope are still to
be agreed.
