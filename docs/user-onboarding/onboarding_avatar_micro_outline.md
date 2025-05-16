# Onboarding: Avatar Selection

## Current Implementation (based on `onboarding-avatar.html`)

- Allows users to select an avatar.
- `rough_draft_outline.md` mentions: "Select Your Avatar" and "Select avatar or Skip (Default Avatar)".
- Links to Flaticon for avatar packs are provided in `rough_draft_outline.md`.

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - `User` model:
        - `avatar_url`:string (could store a path to a pre-defined avatar or a URL if custom uploads were allowed later)
        - `selected_avatar_identifier`:string (if using a set of predefined avatars)
    - `Avatar` model (if managing a predefined set of avatars):
        - `name`:string
        - `image_path`:string
        - `category`:string (optional)
- **Controllers:**
    - `Onboarding::AvatarsController#new` to display avatar options.
    - `Onboarding::AvatarsController#create` (or an update to the user record) to save the avatar choice.
- **Database:**
    - `users` table updated with avatar preference.
    - `avatars` table if managing a predefined set.

### Frontend (Tailwind CSS)

- UI to display a grid or carousel of avatar options.
- A clear "Skip" option that assigns a default avatar.
- Tailwind CSS for styling.
- Display selected avatar prominently.

### Testing (RSpec)

- Request specs for `Onboarding::AvatarsController` (if it's a separate step).
- Model specs for `User` (avatar attribute) and `Avatar` (if created).
- Feature spec for selecting an avatar or skipping.

## Notes/Todos:

- Curate a set of default avatars based on the Flaticon links or other sources.
- Decide on the mechanism for storing/referencing avatars (paths, identifiers).
- Implement the "Skip" functionality and define the default avatar.
- Future consideration: Allow users to upload custom avatars (would require file storage like Active Storage).
