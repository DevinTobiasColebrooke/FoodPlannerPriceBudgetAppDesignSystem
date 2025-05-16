Food Planner Budget App

Outline:

Flows
Onboarding - Disclosure / estimates
Planning
Main Dashboard
Calendar Dashboard
Download report end of month.
Settings
Shopping

Food 
Menu
Allergies
Pull Nutrition data from https://fdc.nal.usda.gov/
Spreadsheet: https://docs.google.com/spreadsheets/d/1JpqsHy5-WDEuXRkeM63dR-vjXAQqYw0LQEKrNLedOOg/edit?gid=5756651#gid=5756651

Cooking/Meal Prep
Tools that are required
How much prep
How much cook time
Eat time
There is the set meal, did you add anything extra?

Shopping
Pull user location or pull user region
Use google maps api/search to get local stores
Search and compile local prices
Account for distance
Would the user like to shop the cheapest, convenient, or a bit of both.
Factor costs based on criteria


UI Design:
	Icons:
https://www.flaticon.com/packs/fruits-and-vegetables-88
https://www.flaticon.com/packs/healthy-food-2
https://www.flaticon.com/packs/italian-food-42
https://www.flaticon.com/packs/cheese-3?word=cheese
https://www.flaticon.com/packs/bubble-tea-2?word=tea
https://www.flaticon.com/packs/vegan-62?word=food
https://www.flaticon.com/search?word=algae&shape=lineal-color
Shield for data - https://www.flaticon.com/free-icon/shield_543116?related_id=543065&origin=search
Calendar Icon -
https://www.flaticon.com/packs/time-and-date-71?word=calendar

Profile avatars:
https://www.flaticon.com/packs/avatar-16?word=profile

Font:
Poppins and oddly calming

Design: mobile first then desktop - Following tailwind design

Tooltips and excerpts will need citations, Gillian McKeith, U.S. Department of Agriculture, Agricultural Research Service, Beltsville Human Nutrition Research Center. FoodData Central. [Internet].[cited (enter date)]. Available from https://fdc.nal.usda.gov/


Tables:
Meals
	Ingredients
	Dishes need
	Price of meal
	Total calories
	Vitamin measurement

User
	Location: Region/Coordinates
	Goals
	Meal difficulty
	Shopping dificulty

Individual food item
	Get from the usda website
	In season: boolean




Flows

Onboarding

Title screen 
Logo top of screen
2 buttons in the middle
Start my journey > User onboarding
Sign in > Log in page > Straight to user dashboard
Bottom privacy policy, terms of service
Security/Data disclosure + security promise - What we do and don’t do
Goals. In the future have a main and optional side goal
How many people - This affects pricing.
1-4
5-10
Option to invite others. Option to download individual reports, option to download all reports at one time. Pdf obviously. 
Option for invited person to set up their own allergies or have the main person set it up.
Any food sensitivities/allergies we should be made aware of?
Dishes available
Instapot, blender, food processor, strainer, juicer
Time spent cooking/prep
Difficulty
How long it takes, the process etc. 
Example: Beans in a can vs dried beans, Dry beans: instapot cooker or no cooker.
Shopping
Set Difficulty
Convenience - going place to place, or majority in one place
Set Budget per week - Flexible yes or no - tooltip, we will try our best to fit the menu within your budget, by enabling flexible we may find ourselves a few dollars over our goal budget.
Use Location or set region
By using your location we can scan the surrounding stores and give you the best price for groceries. If you give us by region we can only give you general estimates of what things may cost. You may change your preference anytime in user settings.  
Select Your Avatar
Select avatar or Skip (Default Avatar)
User is brought to dashboard

Food

Cooking/Meal Prep

Shopping
Business logic
We need to see if the user will allow us to use their location or set their region.

Main Dashboard
	Timer
		Multiple colors around circle timer, the font under gives info like until prep time, until cook time, until eat time.
	
	Under Timer
		Meals like Pre-breakfast, breakfast etc. Current meal or about to be current meal is largest font, further meals away are smaller. The current meal or about to be current meal has tidbit carousel under it in subtitle font. Tidbit carousel includes and displays one at a time: Prep time, cook time, Estimated Calorie count per serving, protein count per serving, this meal is 54% of your daily recommended vitamins, sweet/savory, household favorite.  

	Selecting a meal
		When the user selects a meal title from under the timer, the title goes to the top of the screen as the main dashboard is fading away. This brings up the current meal dashboard

Emergency Button
		Button user can press when they will be rendered imobile for some time. The menu updates with low maintenance easy prep meals, to fight off sickness, 

If easy or medium eating difficulty there is a button/eyebrow that says “I want a dessert”, user is able to filter what type of desert they want and the recipe for it.

There is a eyebrow that says “I’m going shopping” on the days that shopping is not identified.

So current eyebrows are “Im going shopping”, “emergency!”, “I want a dessert”, “Add a snack.”

End Of Month/Start of Month - New eyebrow appears with alert icon on it, “Download my Data”

Current Meal Dashboard
	Title of current meal at the top of the dashboard, stats appear:
	Main course: 
	Side course:
	Nutrition Stats:
	Prep Time:
	Cook Time:
	All meal ingredients are listed
	Superfood highlights
Tool Tips

Close button is at the bottom middle of screen.

Calendar

User Settings
Update goals
Update allergies
Update dishes
Update payment information
Manage Subscription
Download my Data - Statement of meals and calories for each month, user can download pdf.
FAQ
Privacy policy
Terms of Service


Shopping

Meals


Food
Api Guide - https://fdc.nal.usda.gov/api-guide
Data Type Documentation - https://fdc.nal.usda.gov/data-documentation
Cooking/Meal Prep


Shopping
Business logic
If we identify wholesale food stores like costco and sam’s club we may only be able to get an estimate of chicken prices $25-$31, etc. 


Design Systems

Notes:
Need to get user’s local time
Can put what day user will shop and mark on calendar



First 1. Create Design system from playraffleparty.com

2nd. Generate Tech implementation plan
Ai prompt
 Rails 8 app 
Tailwind css
Postresql
Rspec

The design will be mobile focused first.
3 Day free trial
Pricing $10/month, or Annual Subscription for $2.59/month

Break down the app into sections
Macro and micro
And tables w attributes
Attempt to Stick to Dry
No micro implementation plan should expect to be more than 200 lines of code

