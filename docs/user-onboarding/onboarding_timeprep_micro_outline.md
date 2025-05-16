# Onboarding: Time & Prep Difficulty

## Current Implementation (based on `onboarding-timeprep.html`)

- Allows users to specify their preferences for time spent on cooking/prep and meal difficulty.
- `rough_draft_outline.md` mentions: "Time spent cooking/prep", "Difficulty", and an example: "Beans in a can vs dried beans, Dry beans: instapot cooker or no cooker."

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - `User` model:
        - `preferred_prep_time_max_minutes`:integer (or a category like 'short', 'medium', 'long')
        - `preferred_cook_time_max_minutes`:integer (or a category)
        - `meal_difficulty_preference`:string (e.g., 'easy', 'medium', 'hard', or 1-5 scale)
- **Controllers:**
    - `Onboarding::TimePrepController#new` to display options.
    - `Onboarding::TimePrepController#create` to save preferences.
- **Database:**
    - `users` table updated with these preferences.

### Frontend (Tailwind CSS)

- UI for selecting prep time, cook time, and difficulty (e.g., sliders, radio buttons).
- Provide clear explanations for what each difficulty level entails.
- Tailwind CSS for styling.

### Testing (RSpec)

- Request specs for `Onboarding::TimePrepController`.
- Model specs for `User` (validations for these new attributes).
- Feature spec for setting time and difficulty preferences.

## Notes/Todos:

- Define specific categories or ranges for time (e.g., Prep: <15min, 15-30min, >30min).
- Clearly define what constitutes 'easy', 'medium', 'hard' difficulty in terms of skills, steps, or ingredients.
- The example about beans implies the system will suggest recipes based on these preferences, which is a core app logic. 