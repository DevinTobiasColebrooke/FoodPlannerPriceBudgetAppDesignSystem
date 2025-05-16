# Onboarding: Allergies

## Current Implementation (based on `onboarding-allergies.html`)

- Allows users to specify food sensitivities or allergies.
- Likely a multi-select list or checklist.

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - `User` model.
    - `Allergy` model:
        - `name`:string (e.g., 'Peanuts', 'Gluten', 'Dairy')
        - `description`:text (optional)
    - `UserAllergy` join model:
        - `user_id`:integer (foreign key)
        - `allergy_id`:integer (foreign key)
        - `severity`:string (optional, e.g., 'mild', 'moderate', 'severe')
        - `notes`:text (optional, for specific user notes about the allergy)
- **Controllers:**
    - `Onboarding::AllergiesController#new` to display allergy options.
    - `Onboarding::AllergiesController#create` to save the user's allergies.
- **Database:**
    - `allergies` table (could be pre-populated with common allergies).
    - `user_allergies` table.

### Frontend (Tailwind CSS)

- User-friendly interface for selecting multiple allergies (e.g., checkboxes, tags input).
- Option to add custom allergies if not in the predefined list.
- Tailwind CSS for styling.

### Testing (RSpec)

- Request specs for `Onboarding::AllergiesController`.
- Model specs for `User`, `Allergy`, and `UserAllergy`.
- Feature spec for selecting/adding allergies.

## Notes/Todos:

- Compile a comprehensive list of common allergies.
- Decide if users can add custom allergies or only pick from a list.
- Consider if severity levels or notes for allergies are needed at this stage.
- The `rough_draft_outline.md` mentions: "Option for invited person to set up their own allergies or have the main person set it up." This implies allergies might be managed per household member if that feature is built. 