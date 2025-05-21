# AminoAcidsService

## Input
- Age
- Sex
- Weight (for g/kg recommendations)
- Pregnancy/lactation status

## Responsibility
Determine EARs and RDAs for the nine indispensable amino acids (Histidine, Isoleucine, Leucine, Lysine, Methionine + Cysteine, Phenylalanine + Tyrosine, Threonine, Tryptophan, Valine)

## Output
```typescript
{
  histidine: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  isoleucine: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  leucine: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  lysine: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  methionine_cysteine: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  phenylalanine_tyrosine: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  threonine: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  tryptophan: {
    rda_mg_kg: number,
    rda_mg_total: number
  },
  valine: {
    rda_mg_kg: number,
    rda_mg_total: number
  }
}
```

## Dependencies
- DriLookupService (for specific amino acid DRI tables) 