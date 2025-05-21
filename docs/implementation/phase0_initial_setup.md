# Phase 0: Initial Setup & Configuration

## Verify Rails Application

1. Confirm the application was generated with:
```bash
rails new FoodPlannerPriceBudgetApp --skip-test --css tailwind --database postgresql
```

2. Ensure Bundler has installed all gems:
```bash
bundle install
```

## Database Setup

1. Create the development and test databases:
```bash
bin/rails db:create
```

2. Verify `config/database.yml` is correctly configured for PostgreSQL setup.

## Authentication Review

The application includes authentication:
- `app/controllers/concerns/authentication.rb`
- `SessionsController`
- `PasswordsController`
- `User` model
- `Session` model

## Set Root Route

Configure initial root route in `config/routes.rb`:

```ruby
Rails.application.routes.draw do
  # Authentication routes
  resource :session
  resources :passwords, param: :token

  # Define root route
  root "sessions#new" # Or a dedicated welcome controller

  get "up" => "rails/health#show", as: :rails_health_check
end
``` 