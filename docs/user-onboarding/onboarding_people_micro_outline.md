# Onboarding: People

## Current Implementation (based on `onboarding-people.html`)

- Allows the user to specify the number of people the meal plan is for.
- `rough_draft_outline.md` suggests categories: "1-4", "5-10".
- Also mentions: "Option to invite others. Option to download individual reports, option to download all reports at one time. Pdf obviously. Option for invited person to set up their own allergies or have the main person set it up."

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - `User` model:
        - `household_size_category`:string (e.g., '1-4', '5-10') or `number_of_people`:integer.
    - `HouseholdMember` (if inviting others is implemented):
        - `user_id`:integer (foreign key to the main user creating the household)
        - `invited_email`:string
        - `status`:string (e.g., 'pending', 'accepted')
        - `can_set_allergies`:boolean (defaulting to what the main user prefers)
        - `linked_user_id`:integer (foreign key to User table, if the invited person creates an account)
- **Controllers:**
    - `Onboarding::PeopleController#new` to display options.
    - `Onboarding::PeopleController#create` to save household size and potentially initiate invitations.
- **Database:**
    - `users` table updated with household information.
    - `household_members` table (if invitations are implemented).

### Frontend (Tailwind CSS)

- UI for selecting household size category.
- If invitations are implemented, a section to invite others by email.
- Clear indication of report download options (though this might be a post-onboarding feature).

### Testing (RSpec)

- Request specs for `Onboarding::PeopleController`.
- Model specs for `User` (household attributes) and `HouseholdMember` (if applicable).
- Feature spec for selecting household size and inviting members.

## Notes/Todos:

- Decide if `household_size` is a category or a specific number.
- Detail the invitation flow: how are invites sent, accepted, and how do invited users interact?
- Clarify if allergy setup for invited members is part of onboarding or a later setting.
- The report download functionality is likely a core app feature, not just onboarding. 