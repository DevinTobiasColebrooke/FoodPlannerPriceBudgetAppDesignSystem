# ChlorideService
1.  **`calculate_ai_for_adequacy(age, sex, pregnancy_status, lactation_status)`**
    *   **Logic Source:** Chapter 7 ("Chloride: Dietary Reference Intakes for Adequacy").
    *   **Methodology:**
        *   The AIs are based on molar equivalents to sodium AIs, using the molecular weights (Na=23, Cl=35.5).
        *   **Infants (0-12 months):** Based on estimated chloride intake from breast milk and complementary foods.
            *   0-6 months: 0.18 g/d
            *   7-12 months: 0.57 g/d
        *   **Children & Adolescents (1-18 years):** Based on molar equivalents to sodium AIs.
            *   1-3 years: 1.5 g/d
            *   4-8 years: 1.9 g/d
            *   9-13 years: 2.3 g/d
            *   14-18 years: 2.3 g/d
        *   **Adults (≥ 19 years):** Based on molar equivalents to sodium AIs.
            *   All adults ≥ 19y: 2.3 g/d
        *   **Pregnancy & Lactation:** Same as nonpregnant, nonlactating counterparts.
    *   **Output:** `{ ai_g: X }`
    *   **Dependencies:** `DriLookupService` (for pre-calculated values from tables).

2.  **`calculate_ul_for_toxicity(age, sex)`**
    *   **Logic Source:** Chapter 8 ("Chloride: Dietary Reference Intakes for Toxicity").
    *   **Methodology:** Based on molar equivalents to sodium ULs.
        *   **Adults (19-70 years and >70 years):** 3.6 g/d
        *   **Children & Adolescents (1-18 years):** Extrapolated from adult UL.
            *   1-3 years: 1.9 g/d
            *   4-8 years: 2.3 g/d
            *   9-13 years: 2.8 g/d
            *   14-18 years: 3.6 g/d
    *   **Output:** `{ ul_g: X }`
    *   **Dependencies:** `DriLookupService` (for pre-calculated values from tables).

3.  **`calculate_cdrr(age, sex)`**
    *   **Logic Source:** Chapter 9 ("Chloride: Dietary Reference Intakes Based on Chronic Disease").
    *   **Methodology:** The committee concluded that there is **insufficient evidence to establish a chloride CDRR**.
    *   **Output:** `{ cdrr: "Not established", reason: "Insufficient evidence to establish a CDRR." }`
    *   **Dependencies:** None. 