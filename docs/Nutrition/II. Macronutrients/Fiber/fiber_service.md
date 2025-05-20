# Fiber Service

## Overview
The FiberService calculates recommended fiber intake based on age, sex, and energy requirements.

## Inputs
- Age (Years)
- Sex (Male, Female)
- EER (kcal/day)

## Core Logic
1. Calculate AI for Total Fiber
   - Based on g/1000 kcal recommendation
   - Uses DriLookupService for AI values

## Output
```json
{
  "ai_g": number
}
```

## Dependencies
- DriLookupService: For AI values
- EnergyRequirementService: For EER to calculate fiber intake 