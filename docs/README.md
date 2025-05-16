# Project Documentation

This folder contains the documentation for the project. 

Concerned with three things

The user
    What they can eat
    when they can eat again
        Special cases:
            The Athlete - Emphasis on protien intake - Timer on next protien intake
            Diabetic - Emphasis on next time to eat sugar
    Meal Plan
        How serious are they about it
            Super Serious
                If they dislike a bunch of meals we could recommend they drop down a level
                Super Serious has a modal of a pledge - Phone notification 'Take your Pledge' - I pledge/vs not today < this breaks the streak
                    At the end of the day - affirm < increases the streak / Lets give ourself a fresh start tomorrow < Breaks streak
            Lets start out easy
                Introduce them to foods they are okay with



The meals
    The ingredients
        - The nutrion facts of the ingredients - FNDDS For food surveys https://www.ars.usda.gov/northeast-area/beltsville-md-bhnrc/beltsville-human-nutrition-research-center/food-surveys-research-group/docs/fndds-download-databases/ | https://fdc.nal.usda.gov/food-search?type=Survey%20(FNDDS)
        - Able to substitue ingredients
            - Currently Existing in pantry
            - Cheaper option
            - Just different option
    Recipes
        - https://www.nutrition.gov/recipes
        - https://www.myplate.gov/myplate-kitchen
        - https://snaped.fns.usda.gov/resources/recipes-and-menus/snap-ed-recipes
        - https://www.nutrition.gov/topics/shopping-cooking-and-meal-planning/recipe-collection
        - https://www.nutrition.gov/recipes/search?f%5B0%5D=season%3AWinter


The shopping
    The price
    The convienence




Features

Main Dashboard
    Buttons for alternative/impromptu actions
        - I'm Going Shopping - Button present on the days shopping is not scheduled
        - I want Desert - Desert is not offered in meal plan user has to add it. Desert suggestions based on time of day clicked - Earlier in the day      longer deserts are suggested. Toward end of day/meal - quick deserts are suggested.
        - Add a Snack - User can add to their daily meal plan - Modal appears with options, set serving size etc.
        - Emergency! - When user is sick or rendered immobile - Menu will be replaced with quick/easy options
    Timer
        - Time till prep time, then goes into cook time, when cook time ends timer will change into a prompt asking "Did meal time end?" (Also phone notification), once selected yes timer will reappear
    Meals underneath
        -  