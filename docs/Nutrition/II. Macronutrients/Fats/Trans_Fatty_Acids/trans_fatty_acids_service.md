# TransFattyAcidsService

## Input
None directly for calculation, but context of a healthy diet

## Responsibility
State the recommendation to keep Trans Fatty Acid intake as low as possible (from DGA and DRI reports). There isn't a specific gram or % target other than "as low as possible."

## Output
```typescript
{
  recommendation_text: string
}
```

## Dependencies
- DriLookupService (to fetch this qualitative recommendation) 