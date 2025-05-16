# User Onboarding: Views, Controllers, and Data Flow

This document outlines the proposed views, controllers, and data flow for the user onboarding process based on the HTML prototypes.

## Onboarding Steps & Components

Below is a breakdown of each screen in the onboarding flow:

1.  **Welcome Screen (`index.html`)**
    *   **View:** `WelcomeView`
    *   **Presents:** App name, logo, tagline.
    *   **User Actions:**
        *   "Start My Journey"
        *   "Sign In"
    *   **Controller Logic (handled by `OnboardingNavigationController` or `AuthController`):**
        *   Navigate to Disclosure Screen upon "Start My Journey".
        *   Navigate to Login Screen (external to this flow) upon "Sign In".

2.  **Data Disclosure Screen (`onboarding-disclosure.html`)**
    *   **View:** `DisclosureView`
    *   **Presents:** Information on data usage, links to Privacy Policy & Terms.
    *   **User Actions:**
        *   "I Understand, Let's Go!"
        *   "Back to Welcome Screen"
    *   **Controller Logic (handled by `OnboardingNavigationController`):**
        *   Navigate to Goals Screen upon "I Understand".
        *   Navigate back to Welcome Screen.

3.  **Goals Screen (`onboarding-goals.html`)**
    *   **View:** `GoalsView`
    *   **Presents:** Options for user's main dietary/planning goals.
    *   **Data Collected:**
        *   `mainGoal`: String (e.g., "save_money", "eat_healthier") - *Required*
        *   `sideGoals`: List<String> (e.g., ["reduce_waste", "new_recipes"]) - *Optional*
    *   **User Actions:** "Next", "Back"
    *   **Controller Logic (handled by `GoalsController` + `OnboardingNavigationController`):**
        *   Validate `mainGoal` is selected.
        *   Store `mainGoal` and `sideGoals` in `OnboardingProfile`.
        *   Navigate to Household Size Screen.
        *   Navigate back to Disclosure Screen.

4.  **Household Size Screen (`onboarding-people.html`)**
    *   **View:** `HouseholdSizeView`
    *   **Presents:** Options for the number of people the user is planning for.
    *   **Data Collected:**
        *   `householdSize`: String (e.g., "1-4", "5-10")
    *   **User Actions:** "Next", "Back"
    *   **Controller Logic (handled by `HouseholdSizeController` + `OnboardingNavigationController`):**
        *   Store `householdSize` in `OnboardingProfile`.
        *   Navigate to Allergies Screen.
        *   Navigate back to Goals Screen.

5.  **Allergies & Sensitivities Screen (`onboarding-allergies.html`)**
    *   **View:** `AllergiesView`
    *   **Presents:** Input for custom allergies and quick select for common ones.
    *   **Data Collected:**
        *   `allergies`: List<String> (e.g., ["peanuts", "gluten", "shellfish"])
    *   **User Actions:** "Next", "Back", Add Custom Allergy, Remove Allergy, Select/Deselect Common Allergen.
    *   **Controller Logic (handled by `AllergiesController` + `OnboardingNavigationController`):**
        *   Manage the dynamic list of allergies.
        *   Store `allergies` in `OnboardingProfile`.
        *   Navigate to Equipment Screen.
        *   Navigate back to Household Size Screen.

6.  **Kitchen Equipment Screen (`onboarding-equipment.html`)**
    *   **View:** `EquipmentView`
    *   **Presents:** Checklist of common kitchen equipment.
    *   **Data Collected:**
        *   `equipment`: List<String> (e.g., ["instapot", "blender", "oven"])
    *   **User Actions:** "Next", "Back", Select/Deselect Equipment.
    *   **Controller Logic (handled by `EquipmentController` + `OnboardingNavigationController`):**
        *   Store `equipment` in `OnboardingProfile`.
        *   Navigate to Cooking Style Screen.
        *   Navigate back to Allergies Screen.

7.  **Cooking Style Screen (`onboarding-timeprep.html`)**
    *   **View:** `CookingStyleView`
    *   **Presents:** Options for preferred cooking time and meal difficulty.
    *   **Data Collected:**
        *   `cookingTimePreference`: String (e.g., "quick", "moderate", "leisurely", "varies")
        *   `mealDifficultyPreference`: String (e.g., "easy", "medium", "involved")
    *   **User Actions:** "Next", "Back"
    *   **Controller Logic (handled by `CookingStyleController` + `OnboardingNavigationController`):**
        *   Store `cookingTimePreference` and `mealDifficultyPreference` in `OnboardingProfile`.
        *   Navigate to Shopping Preferences Screen.
        *   Navigate back to Equipment Screen.

8.  **Shopping Preferences Screen (`onboarding-shopping.html`)**
    *   **View:** `ShoppingPrefsView`
    *   **Presents:** Options for shopping difficulty, budget, and location preferences.
    *   **Data Collected:**
        *   `shoppingDifficulty`: String (e.g., "convenient", "cheapest", "balanced")
        *   `weeklyBudget`: Double? (Optional, min $15 if specified)
        *   `isBudgetFlexible`: Boolean
        *   `locationPreference`: String (e.g., "auto", "region")
        *   `regionZip`: String? (Conditional on `locationPreference`)
    *   **User Actions:** "Next", "Back"
    *   **Controller Logic (handled by `ShoppingPrefsController` + `OnboardingNavigationController`):**
        *   Validate `weeklyBudget` if entered (must be >= 15).
        *   Handle conditional visibility of `regionZip` input.
        *   Store shopping preferences in `OnboardingProfile`.
        *   Navigate to Avatar Screen.
        *   Navigate back to Cooking Style Screen.

9.  **Avatar Screen (`onboarding-avatar.html`)**
    *   **View:** `AvatarView`
    *   **Presents:** Selection of avatars or option to skip.
    *   **Data Collected:**
        *   `avatarUrl`: String? (Path or identifier to selected avatar, or null/default if skipped)
    *   **User Actions:** "Finish Setup & Go to Dashboard!", "Back", "Skip & Use Default Avatar"
    *   **Controller Logic (handled by `AvatarController` + `OnboardingNavigationController` or `AppNavigationController`):**
        *   Store `avatarUrl` in `OnboardingProfile`.
        *   Finalize the onboarding process (e.g., save `OnboardingProfile` to persistent storage).
        *   Navigate to the main application dashboard.
        *   Navigate back to Shopping Preferences Screen.

## General Controller Concepts

*   **`OnboardingNavigationController`**:
    *   Manages the sequence of onboarding views.
    *   Likely holds a reference to the `OnboardingProfile` data object.
    *   Handles "Next" and "Back" navigation logic between steps.

*   **Step-Specific Controllers** (e.g., `GoalsController`, `AllergiesController`, `ShoppingPrefsController`):
    *   Encapsulate the logic specific to a single onboarding screen.
    *   Responsible for:
        *   Updating the `OnboardingProfile` with data from their view.
        *   Performing any view-specific input validation before proceeding.
        *   Managing local UI state (e.g., dynamically adding allergy tags).
    *   These could be distinct classes or methods within a larger controller or view model, depending on the chosen architecture (MVC, MVVM, etc.).

*   **`AuthController` (Conceptual)**:
    *   Handles overall authentication state.
    *   The "Sign In" action on the `WelcomeView` would likely be routed through this controller.

## Data Model: `OnboardingProfile`

A temporary data object to accumulate user choices throughout the onboarding flow.

*   `mainGoal: String?`
*   `sideGoals: List<String> = []`
*   `householdSize: String?`
*   `allergies: List<String> = []`
*   `equipment: List<String> = []`
*   `cookingTimePreference: String?`
*   `mealDifficultyPreference: String?`
*   `shoppingDifficulty: String?`
*   `weeklyBudget: Double? = null`
*   `isBudgetFlexible: Boolean = true` (default from HTML)
*   `locationPreference: String?`
*   `regionZip: String? = null`
*   `avatarUrl: String? = null` (or a default avatar identifier)

Upon completion of the avatar screen ("Finish Setup"), this `OnboardingProfile` should be processed:
1.  Saved to the user's persistent profile (likely via an API call to a backend service).
2.  The onboarding state should be marked as complete for the user.
3.  The user is then redirected to the main application dashboard.

## Next Steps / Considerations

*   **Error Handling:** Each step should gracefully handle invalid input or API errors if data is submitted progressively.
*   **State Persistence:** If a user quits mid-onboarding, consider if their progress should be saved and resumed.
*   **API Design:** Define API endpoints for creating/updating the user profile with onboarding data.
*   **UI/UX Details:** While the HTML provides a good base, further refinement of interactions, loading states, and feedback messages will be needed during implementation. 