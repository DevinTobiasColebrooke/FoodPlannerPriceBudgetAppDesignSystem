# Onboarding: Disclosure

## Current Implementation (based on `onboarding-disclosure.html`)

- Displays security and data disclosure information.
- Likely contains static text content.

## Future Implementation (Ruby on Rails 8, PostgreSQL, Tailwind CSS, RSpec)

### Backend (Rails/PostgreSQL)

- **Models:**
    - Potentially a `LegalDocument` model if content needs to be versioned or dynamically managed.
        - `title`:string
        - `content`:text
        - `version`:string
        - `document_type`:string (e.g., 'disclosure', 'privacy_policy', 'terms_of_service')
- **Controllers:**
    - `Onboarding::DisclosuresController#show` to display the content.
- **Database:**
    - `legal_documents` table.

### Frontend (Tailwind CSS)

- Style the page according to the design system using Tailwind CSS.
- Ensure responsive design.

### Testing (RSpec)

- Request spec for `Onboarding::DisclosuresController#show`.
- Model spec for `LegalDocument` (if created).
- Feature spec to ensure the page renders correctly and displays the information.

## Notes/Todos:

- Confirm if the disclosure content needs to be dynamic or can remain static.
- The `rough_draft_outline.md` mentions: "Security/Data disclosure + security promise - What we do and don't do". Ensure this content is comprehensive. 