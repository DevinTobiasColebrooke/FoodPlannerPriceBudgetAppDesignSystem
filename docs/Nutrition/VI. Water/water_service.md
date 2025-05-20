# WaterService
1.  **`calculate_ai_for_adequacy(age, sex, pregnancy_status, lactation_status)`**
    *   **Logic Source:** Chapter 2 ("Water: Dietary Reference Intakes for Adequacy").
    *   **Methodology:**
        *   The AIs are based on median total water intakes from NHANES (U.S.) and CCHS Nutrition 2015 (Canada) for apparently healthy individuals.
        *   **Infants (0-12 months):** Based on estimated water intake from breast milk and complementary foods.
            *   0-6 months: 0.7 L/d
            *   7-12 months: 0.8 L/d
        *   **Children & Adolescents (1-18 years):** Based on median intakes from surveys, stratified by age and sex.
            *   1-3 years: 1.3 L/d
            *   4-8 years: 1.7 L/d
            *   9-13 years: 2.4 L/d (males), 2.1 L/d (females)
            *   14-18 years: 3.3 L/d (males), 2.3 L/d (females)
        *   **Adults (≥ 19 years):** Based on median intakes, stratified by sex.
            *   Males ≥ 19y: 3.7 L/d
            *   Females ≥ 19y: 2.7 L/d
        *   **Pregnancy:** Based on nonpregnant AIs plus estimated needs.
            *   14-18y: 2.8 L/d
            *   19-50y: 3.0 L/d
        *   **Lactation:** Based on nonpregnant AIs plus estimated secretion in milk.
            *   14-18y: 3.8 L/d
            *   19-50y: 3.8 L/d
    *   **Output:** `{ ai_liters: X }`
    *   **Dependencies:** `DriLookupService` (for specific AI values from tables).

2.  **`calculate_ul_for_toxicity(age, sex)`**
    *   **Logic Source:** Chapter 3 ("Water: Dietary Reference Intakes for Toxicity").
    *   **Methodology:** "The committee concludes that there is **insufficient evidence of water toxicity risk within the apparently healthy population to establish a water Tolerable Upper Intake Level (UL)**."
    *   **Output:** `{ ul: "Not established", reason: "Insufficient evidence of toxicity risk in apparently healthy population." }`
    *   **Dependencies:** None.

3.  **`calculate_cdrr(age, sex)`**
    *   **Logic Source:** Chapter 4 ("Water: Dietary Reference Intakes Based on Chronic Disease").
    *   **Methodology:** The committee concluded that there is **insufficient evidence to establish a water CDRR**.
    *   **Output:** `{ cdrr: "Not established", reason: "Insufficient evidence to establish a CDRR." }`
    *   **Dependencies:** None. 