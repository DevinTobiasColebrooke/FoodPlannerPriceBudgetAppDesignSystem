# PotassiumService
1.  **`calculate_ai_for_adequacy(age, sex, pregnancy_status, lactation_status)`**
    *   **Logic Source:** Chapter 4 ("Potassium: Dietary Reference Intakes for Adequacy").
    *   **Methodology:**
        *   The 2005 AIs were based on blunting salt sensitivity and reducing kidney stone risk, aiming for 4,700 mg/d for adults.
        *   **This Report (New AIs):** The committee concluded that none of the reviewed indicators for potassium requirements offer sufficient evidence to establish EAR/RDA.
        *   New AIs are based on median usual potassium intakes from NHANES (U.S.) and CCHS Nutrition 2015 (Canada) for *apparently healthy, normotensive individuals without a self-reported history of cardiovascular disease*. The *highest* median intake across the two surveys for each DRI age/sex group was chosen and mathematically rounded.
        *   **Infants (0-12 months):** AIs based on estimated potassium intake from breast milk and complementary foods (see Appendix F).
            *   0-6 months: 400 mg/d
            *   7-12 months: 860 mg/d
        *   **Children & Adolescents (1-18 years):** Extrapolated from adult AI methodology, using the highest median usual intakes. Values are sex-stratified for ages 9+. (See Table 4-3)
        *   **Adults (≥ 19 years):** Stratified by sex. The AIs are **not** further stratified by adult age groups (19-30, 31-50, 51-70, >70 all have the same AI within each sex) because different values would convey greater precision than available. (See Table 4-4)
            *   Males ≥ 19y: 3,400 mg/d
            *   Females ≥ 19y: 2,600 mg/d
        *   **Pregnancy:** Based on nonpregnant AIs plus estimated accretion. (See Table 4-5)
            *   14-18y: 2,600 mg/d
            *   19-50y: 2,900 mg/d
        *   **Lactation:** Based on nonpregnant AIs plus estimated secretion in milk. (See Table 4-6)
            *   14-18y: 2,500 mg/d
            *   19-50y: 2,800 mg/d
    *   **Output:** Potassium AI (mg/day).
    *   **Dependencies:** `DriLookupService` (for specific AI values from new tables like Table 4-7).

2.  **`calculate_ul_for_toxicity(age, sex)`**
    *   **Logic Source:** Chapter 5 ("Potassium: Dietary Reference Intakes for Toxicity").
    *   **Methodology:** "The committee concludes that there is **insufficient evidence of potassium toxicity risk within the apparently healthy population to establish a potassium Tolerable Upper Intake Level (UL)**."
    *   **Output:** `{ ul: "Not established", reason: "Insufficient evidence of toxicity risk in apparently healthy population." }`
    *   **Dependencies:** None.

3.  **`calculate_cdrr(age, sex)`**
    *   **Logic Source:** Chapter 6 ("Potassium: Dietary Reference Intakes Based on Chronic Disease").
    *   **Methodology:** The committee concluded that despite moderate evidence for potassium supplementation reducing blood pressure, limitations (heterogeneity, lack of intake-response, lack of supporting CVD evidence) **prevented the establishment of a potassium CDRR.**
    *   **Output:** `{ cdrr: "Not established", reason: "Insufficient evidence to establish a CDRR, particularly regarding intake-response and direct cardiovascular benefits." }`
    *   **Dependencies:** None.