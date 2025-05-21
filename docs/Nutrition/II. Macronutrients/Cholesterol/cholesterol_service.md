# CholesterolService

## Input
None directly for calculation, but context of a healthy diet

## Responsibility
State the recommendation to keep dietary Cholesterol intake as low as possible while consuming a nutritionally adequate diet (from DGA and DRI reports). No specific mg target is usually set in recent guidelines.

## Output
```typescript
{
  recommendation_text: string
}
```

## Dependencies
- DriLookupService 