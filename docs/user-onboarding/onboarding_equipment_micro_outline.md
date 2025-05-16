# Onboarding: Equipment

## Current Implementation (based on `onboarding-equipment.html`)

- Allows users to select available kitchen equipment.
- `rough_draft_outline.md` lists: "Instapot, blender, food processor, strainer, juicer".

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - `User` model.
    - `Equipment` model:
        - `name`:string (e.g., 'Instapot', 'Blender')
    - `UserEquipment` join model:
        - `user_id`:integer (foreign key)
        - `equipment_id`:integer (foreign key)
- **Controllers:**
    - `Onboarding::EquipmentController#new` to display equipment options.
    - `Onboarding::EquipmentController#create` to save the user's available equipment.
- **Database:**
    - `equipment` table (pre-populated with common kitchen equipment).
    - `user_equipment` table.

### Frontend (Tailwind CSS)

- UI for selecting multiple pieces of equipment (e.g., checkboxes).
- Tailwind CSS for styling.

### Testing (RSpec)

- Request specs for `Onboarding::EquipmentController`.
- Model specs for `User`, `Equipment`, and `UserEquipment`.
- Feature spec for selecting equipment.

## Notes/Todos:

- Finalize the list of equipment to offer as options.
- Consider if users can add custom equipment not on the list. 