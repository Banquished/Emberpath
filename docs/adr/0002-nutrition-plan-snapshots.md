# Save nutrition plans as explicit snapshots

Status: Accepted
Scope: Nutrition service.

A calculation is provisional until the user explicitly saves it. Saving
captures the accepted inputs, calculation method and version, targets,
weekday allocation and overrides as a plan snapshot. Each user has at most
one active plan. A save takes effect immediately, replacing any active
version while retaining it as history. The user can explicitly end the
active plan without deleting its snapshot; a later plan requires a new save.
Future-dated starts, backdating and retroactive edits are outside v1.0.
A changed weight measurement, weight goal or calculation method never
silently changes a saved plan; the user must explicitly recalculate and
save a replacement. Automatically recalculating plans would make past
targets and user choices impossible to explain reliably. This requires
plan versioning and a clear replacement flow; scheduled curves and
permanent deletion remain separate decisions.
