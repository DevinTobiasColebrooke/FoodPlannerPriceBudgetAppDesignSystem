# Onboarding: Goals

## Current Implementation (based on `onboarding-goals.html`)

- Allows users to select or input their goals.
- The `rough_draft_outline.md` mentions: "Goals. In the future have a main and optional side goal". The current HTML likely provides a basic selection.

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - `User` model will have a relationship with `Goal`.
    - `Goal` model:
        - `name`:string (e.g., 'Weight Loss', 'Muscle Gain', 'Healthy Eating')
        - `description`:text (optional)
        - `is_primary`:boolean (to distinguish main vs. side goals)
    - `UserGoal` join model (if a user can have multiple goals):
        - `user_id`:integer (foreign key)
        - `goal_id`:integer (foreign key)
        - `is_primary`:boolean (denormalized for easier querying, or handled via `Goal` model)
        - `custom_goal_details`:text (if users can specify details for a selected goal)
- **Controllers:**
    - `Onboarding::GoalsController#new` to display goal options.
    - `Onboarding::GoalsController#create` to save the user's selected goals.
- **Database:**
    - `goals` table.
    - `user_goals` table (join table).

### Frontend (Tailwind CSS)

- Design a user-friendly interface for selecting/defining goals.
- Use Tailwind CSS for styling.
- Consider how to present main vs. optional side goals clearly.

### Testing (RSpec)

- Request specs for `Onboarding::GoalsController` (new, create).
- Model specs for `User`, `Goal`, and `UserGoal`.
- Feature spec for the goal selection process.

## Notes/Todos:

- Determine the predefined list of goals.
- Clarify how main and optional side goals will be handled in the UI and backend.
- Decide if users can input custom goals or only select from a predefined list. 