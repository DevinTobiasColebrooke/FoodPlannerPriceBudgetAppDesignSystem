# Sodium Service
1.  **`calculate_ai_for_adequacy(age, sex, pregnancy_status, lactation_status)`**
    *   **Logic Source:** Chapter 8 ("Sodium: Dietary Reference Intakes for Adequacy").
    *   **Methodology:**
        *   EAR/RDA not established due to insufficient intake-response data.
        *   The 2005 AIs were based on meeting nutrient needs and covering sweat losses.
        *   **This Report (New AIs):** The adult AI (1,500 mg/d for 19-50y) was based on the lowest levels of sodium intake in randomized trials and the best-designed balance study (Allsopp et al., 1998) that showed neutral or positive balance without deficiency symptoms. This is a shift from using population median intakes.
        *   **Infants (0-12 months):** Based on estimated sodium intake from breast milk. (See Table 8-3)
            *   0-6 months: 110 mg/d
            *   7-12 months: 370 mg/d
        *   **Children & Adolescents (1-18 years):** Extrapolated from the adult AI of 1,500 mg/d using rounded average sedentary EERs (see Table 8-5). This is a change from the 2005 report.
        *   **Adults (≥ 19 years):** The AI for adults ≥19 years is **1,500 mg/d**. The committee did not extrapolate for older adults due to limited evidence (see Table 8-7).
        *   **Pregnancy & Lactation:** No evidence suggests needs differ from nonpregnant, nonlactating counterparts. AI is 1,500 mg/d (Tables 8-8, 8-9).
    *   **Output:** Sodium AI (mg/day).
    *   **Dependencies:** `DriLookupService` (for specific AI values from new tables like Table 8-10).

2.  **`calculate_ul_for_toxicity(age, sex)`**
    *   **Logic Source:** Chapter 9 ("Sodium: Dietary Reference Intakes for Toxicity").
    *   **Methodology:** "The committee concludes that there is **insufficient evidence of sodium toxicity risk within the apparently healthy population to establish a sodium Tolerable Upper Intake Level (UL)**." The previous UL was based on blood pressure, which is now considered under CDRR.
    *   **Output:** `{ ul: "Not established", reason: "Insufficient evidence of *toxicological* risk. Chronic disease risk is addressed by CDRR." }`
    *   **Dependencies:** None.

3.  **`calculate_cdrr(age, sex, pregnancy_status, lactation_status)`**
    *   **Logic Source:** Chapter 10 ("Sodium: Dietary Reference Intakes Based on Chronic Disease").
    *   **Methodology:**
        *   **The CDRR is the intake above which intake reduction is expected to reduce chronic disease risk within an apparently healthy population.**
        *   Based on a synthesis of evidence for cardiovascular disease, hypertension, systolic blood pressure, and diastolic blood pressure.
        *   Moderate to high strength of evidence for causal relationship and intake-response between sodium and these indicators.
        *   **Adults (19-70 years and >70 years):** The CDRR is **2,300 mg/d**. This means reducing sodium intake *if it is above 2,300 mg/d* is expected to reduce chronic disease risk. (See Box 10-8, Table 10-14).
        *   **Children & Adolescents (1-18 years):** Extrapolated from the adult CDRR of 2,300 mg/d using rounded average sedentary EERs. (See Table 10-14)
            *   1-3 years: Reduce if >1,200 mg/d
            *   4-8 years: Reduce if >1,500 mg/d
            *   9-13 years: Reduce if >1,800 mg/d
            *   14-18 years: Reduce if >2,300 mg/d
        *   **Pregnancy & Lactation:** Same CDRR as their nonpregnant, nonlactating age group counterparts (i.e., 2,300 mg/d if ≥19y, or the age-specific CDRR if <19y).
    *   **Output:** `{ cdrr_mg: X, cd_risk_reduction_instruction: "Reduce if above X mg/day" }`
    *   **Dependencies:** `DriLookupService` (for specific CDRR values from new tables like Table 10-14).