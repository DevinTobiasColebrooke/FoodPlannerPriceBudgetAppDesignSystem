# SaturatedFatService

## Input
- EER

## Responsibility
Calculate the limit for Saturated Fatty Acids (typically <10% of EER, from DGA)

## Output
```typescript
{
  saturated_fat_limit_percent: number,
  saturated_fat_limit_g: number
}
```

## Dependencies
- EnergyRequirementService
- DriLookupService (to confirm DGA recommendation) 