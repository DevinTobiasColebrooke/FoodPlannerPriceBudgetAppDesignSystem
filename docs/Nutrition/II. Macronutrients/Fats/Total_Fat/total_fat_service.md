# TotalFatService

## Input
- Age
- Sex
- EER

## Responsibility
- Calculate AI for Total Fat (infants only, g/day)
- Calculate AMDR for Total Fat (% of EER, and g/day)

## Output
```typescript
{
  total_fat_ai_g_infant: number,
  total_fat_amdr_percent_min: number,
  total_fat_amdr_percent_max: number,
  total_fat_amdr_g_min: number,
  total_fat_amdr_g_max: number
}
```

## Dependencies
- DriLookupService
- EnergyRequirementService 