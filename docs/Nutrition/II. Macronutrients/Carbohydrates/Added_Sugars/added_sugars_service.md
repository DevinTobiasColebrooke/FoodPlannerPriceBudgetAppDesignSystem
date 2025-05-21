# AddedSugarsService

## Input
- EER (from EnergyRequirementService)

## Responsibility
Calculate the limit for Added Sugars (typically <10% of EER, from DGA)

## Output
```typescript
{
  added_sugars_limit_percent: number,
  added_sugars_limit_g: number
}
```

## Dependencies
- EnergyRequirementService
- DriLookupService (to confirm the DGA recommendation if it's stored there) 