# Preserve planned weekly calories when allocating weekdays

Status: Accepted
Scope: Nutrition plans.

The calculation supplies a chosen average daily calorie target; the plan owns
its repeating seven-day allocation. Its planned weekly calorie total is seven
times that chosen target, and the seven weekday values must sum to it. Moving
calories between days must not silently change either another day's target or
the weekly total. Show the remaining allocation and require a balanced week
before saving. Changing the overall target instead creates an explicit new
plan version. Allowing independent day edits to change the weekly total would
make a weekday/weekend redistribution silently alter the user's chosen
target. The [nutrition PRD](../nutrition-service/prd.md) defines daily
nutrient derivation separately; this ADR concerns calorie conservation.
