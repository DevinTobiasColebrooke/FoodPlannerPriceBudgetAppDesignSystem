# Phase 0: Initial Setup & Configuration

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