# Phase 2: User Onboarding

## Onboarding Namespace & Base Controller

Generate controllers for each onboarding step:

```bash
bin/rails g controller onboarding/Welcome show
bin/rails g controller onboarding/Disclosures show update
bin/rails g controller onboarding/Goals new create
# ... and so on for each onboarding step controller
```

## Routes for Onboarding

```ruby
# config/routes.rb
namespace :onboarding do
  resource :welcome, only: [:show]
  resource :disclosure, only: [:show, :update]
  resource :profile_info, only: [:new, :create]
  resource :goal, only: [:new, :create]
  resource :people, only: [:new, :create]
  resource :allergy, only: [:new, :create]
  resource :equipment, only: [:new, :create]
  resource :time_prep, only: [:new, :create]
  resource :shopping, only: [:new, :create]
  resource :avatar, only: [:new, :create]
  resource :finalize, only: [:create]
end
```

## Onboarding Profile Form Object

```ruby
# app/forms/onboarding_profile.rb
class OnboardingProfile
  include ActiveModel::Model
  include ActiveModel::Attributes

  attribute :main_goal, :string
  attribute :side_goals, :string_array
  attribute :household_size, :string
  attribute :avatar_url, :string
  # ... other attributes
end
```

## Base Controller

```ruby
# app/controllers/onboarding/base_controller.rb
class Onboarding::BaseController < ApplicationController
  layout "onboarding"

  private

  def onboarding_profile
    @onboarding_profile ||= OnboardingProfile.new(session[:onboarding_profile] || {})
  end

  def update_onboarding_profile(params)
    profile_data = onboarding_profile.attributes.merge(params)
    session[:onboarding_profile] = profile_data
    @onboarding_profile = OnboardingProfile.new(profile_data)
  end
end
```

## Example Controller Implementation

```ruby
# app/controllers/onboarding/goals_controller.rb
class Onboarding::GoalsController < Onboarding::BaseController
  def new
    @profile = onboarding_profile
    @goals = Goal.all
  end

  def create
    update_onboarding_profile(goal_params)

    if @onboarding_profile.valid?
      redirect_to new_onboarding_people_path
    else
      @goals = Goal.all
      render :new, status: :unprocessable_entity
    end
  end

  private

  def goal_params
    params.require(:onboarding_profile).permit(:main_goal, side_goals: [])
  end
end
```

## Finalize Onboarding

```ruby
# app/controllers/onboarding/finalize_controller.rb
class Onboarding::FinalizeController < Onboarding::BaseController
  def create
    profile = onboarding_profile
    user = current_user

    user.assign_attributes(
      household_size: profile.household_size,
      cooking_time_preference: profile.cooking_time_preference,
      meal_difficulty_preference: profile.meal_difficulty_preference,
      avatar_url: profile.avatar_url,
      onboarding_completed_at: Time.current
    )

    # Create associated records
    if profile.main_goal.present?
      goal = Goal.find_by(name: profile.main_goal)
      user.user_goals.create(goal: goal, is_primary: true) if goal
    end

    profile.side_goals&.each do |side_goal_name|
      goal = Goal.find_by(name: side_goal_name)
      user.user_goals.create(goal: goal, is_primary: false) if goal
    end

    # ... handle other associations ...

    if user.save
      session.delete(:onboarding_profile)
      redirect_to authenticated_root_path, notice: "Welcome! Your profile is set up."
    else
      redirect_to new_onboarding_avatar_path, alert: "Could not save profile. Please review your entries."
    end
  end
end
```

For detailed controller implementations, see `controllers/onboarding.md`. 