# Phase 3: Core Application Features

## Dashboard

Generate the dashboard controller:
```bash
bin/rails g controller Dashboard show
```

Set up authenticated root route:
```ruby
# config/routes.rb
authenticated :user do
  root 'dashboard#show', as: :authenticated_root
end
root "sessions#new" # if not authenticated
```

## Meal/Recipe Display

Generate recipe controller:
```bash
bin/rails g controller Recipes show
```

## Calendar

Generate meal plans controller:
```bash
bin/rails g controller MealPlans index
```

## Shopping Lists

Generate shopping list controllers:
```bash
bin/rails g controller ShoppingLists index show new create edit update destroy
bin/rails g controller ShoppingListItems create update destroy
```

## Settings

Generate settings controller:
```bash
bin/rails g controller Settings show edit update
```

For detailed controller implementations, see `controllers/dashboard.md`.

## Feature Implementation Details

### Dashboard
- Display upcoming meals
- Show user stats
- Circular timer implementation with JavaScript

### Recipe Display
- Show recipe details
- Display ingredients
- Show nutrition information
- Recipe instructions

### Calendar
- Monthly/weekly view
- Meal plan entries
- Drag and drop functionality
- JavaScript calendar implementation

### Shopping Lists
- Create and manage lists
- Add items from recipes
- Store preferences
- Price tracking (future feature)

### Settings
- Profile management
- Preferences
- Dietary restrictions
- Equipment settings

For detailed view implementations, refer to the HTML prototypes in the design system. 