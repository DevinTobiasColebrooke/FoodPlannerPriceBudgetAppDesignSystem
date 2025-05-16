# Onboarding: Shopping Preferences

## Current Implementation (based on `onboarding-shopping.html`)

- Allows users to set shopping difficulty, budget, and location preferences.
- `rough_draft_outline.md` details:
    - "Set Difficulty: Convenience - going place to place, or majority in one place"
    - "Set Budget per week - Flexible yes or no - tooltip..."
    - "Use Location or set region - By using your location we can scan the surrounding stores... If you give us by region we can only give you general estimates..."

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - `User` model:
        - `shopping_difficulty_preference`:string (e.g., 'convenience_focused', 'single_store_preferred')
        - `weekly_budget_amount`:decimal
        - `budget_is_flexible`:boolean
        - `location_preference_type`:string (e.g., 'use_device_location', 'set_region')
        - `shopping_region`:string (if `location_preference_type` is 'set_region')
        - `latitude`:decimal (if `location_preference_type` is 'use_device_location' and permission given)
        - `longitude`:decimal (if `location_preference_type` is 'use_device_location' and permission given)
- **Controllers:**
    - `Onboarding::ShoppingController#new` to display options.
    - `Onboarding::ShoppingController#create` to save preferences.
- **Services (Potential):
    - `LocationService` to handle geocoding if a region is provided or to process device location.
- **Database:**
    - `users` table updated with shopping preferences.

### Frontend (Tailwind CSS)

- UI for selecting shopping difficulty.
- Input for weekly budget and a toggle for flexibility (with tooltip).
- Options to allow location access or manually set a region (e.g., ZIP code, city/state).
- Tailwind CSS for styling.
- JavaScript for handling location API if `use_device_location` is chosen.

### Testing (RSpec)

- Request specs for `Onboarding::ShoppingController`.
- Model specs for `User` (validations for shopping attributes).
- Feature specs for setting shopping preferences, including location permission handling if applicable.
- Service spec for `LocationService` if created.

## Notes/Todos:

- Define clear options for "Shopping Difficulty".
- Detail the implementation for "Use Location" vs. "Set Region". What level of granularity for region (ZIP, city, state)?
- The actual scanning of local stores and price compilation is a major post-onboarding feature relying on external APIs (e.g., Google Maps API, grocery store APIs if available).
- The tooltip for budget flexibility needs to be implemented. 