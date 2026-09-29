# Keep nutrition calculation and plans in one deployable

Status: Accepted
Scope: Nutrition service; future cross-service boundaries remain open.

The initial nutrition service has a calculation module that owns estimation
and nutrient-derivation rules and a plan module that owns saved-plan lifecycle
and weekday target allocation. Food and activity logging are outside its
first release.
A calculator service and a separate
plan service could deploy independently, but would introduce a network and
authorization contract, coordinated versioning, and additional operational
failure modes without a current need for independent scaling or release
cadence. Keep the modules independently understandable and testable without
adding pass-through abstractions solely to enable a hypothetical split. Revisit
deployment boundaries when ownership, security or operational needs justify
them; this decision does not assign future logging to a particular service.
