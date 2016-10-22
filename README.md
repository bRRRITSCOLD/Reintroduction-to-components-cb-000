# Reintroduction to Components

## Overview
Let's set the stage for a second: you're on Netflix and would like to re-watch your favorite episode of The Office. You click 'Search' and begin to type in "the o". As you type each character Netflix filters down the shows until all you see is Michael Scott looking pensive. (insert picture). We know how to accomplish this same feat. with vanilla React - we would set the state of our parent component, add a function that checked for user input, then pass down the callback function as props into a child component. In redux our interaction with state inside components is a bit different...

## Objectives
1. Explain how components subscribe to store
2. Describe how a user's interactions with a component trigger updates to data
3. Describe how data gets back to components

## Outline
1. Show how components access store
  - MapStateToProps
  - MapDispatchToProps
2. Demonstrate that class components re-render when state is changed
3. Show presentational (functional) components that do not interact directly with state (only have data passed down as props)
4. .dispatch() is the way to trigger a state change
5. .subscribe(listener) listens for .dispatch(). Called anytime an action is dispatched
6. Build out a simple YouTube search to demonstrate how state is passed around and components are re-rendered on state change
