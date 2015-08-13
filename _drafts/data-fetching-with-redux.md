---
title: Data Fetching with Redux
---

A common saying about React is that it 'is just the V in MVC'. This becomes
evident when you start trying to manage data fetching and and data dependencies
in larger React applications. Flux is the architecture that enables building these
larger complex applications while easily managing state and keeping your
React components in sync with your data.

## Redux

One Flux library that has been gaining a ton of momentum in the community is
[Redux][redux], and it is the library used in the examples below.

The Redux docs do an amazing job of describing the basics of Redux, so please
go read [the Gist of Redux][the-gist] before continuing.

## Reducer

First, we have to define our reducer, the pure function that modifies the state
based on actions. The search reducer will be responsible for adding new suggestions
when it receives a `ADD_SUGGESTIONS` action. Also, the reducer should track
the current search term using the `UPDATE_SEARCH` action.

Remember, reducers must be pure functions. They must not modify the passed state!
In the example below the state is copied into a new object and just the relevant
bit is overwritten in the new object.

{% highlight js %}
const initialState = {
  suggestions: [],
  search: ''
};

export default function search(state = initialState, action) {
  switch(action.type) {
  case 'ADD_SUGGESTIONS':
    return Object.assign({}, state, { suggestions: action.suggestions });
  case 'UPDATE_SEARCH':
    return Object.assign({}, state, { search: action.search });
  default:
    return state;
  }
}
{% endhighlight %}

## Actions

Actions are simply plain objects that by convention have a type (i.e. `ADD_SUGGESTIONS`,
`UPDATE_SEARCH`) and some data relevant to the action. The reducer above expects
the `ADD_SUGGESTIONS` action to have an array in the `suggestions` property and the `UPDATE_SEARCH`
action to have a `search` property with a string.

The general approach is to define **action
creators**, functions that return actions, to make it easier to change actions
in the future. So, let's make the two action creators.

{% highlight js %}
export function updateSearch(search) {
  return {
    type: 'UPDATE_SEARCH',
    search: search
  }
}

// addSuggestions returns an async action creator
export function addSuggestions(search) {
  return dispatch => {
    fetch(`${GITHUB_API}`)
      .then(res => res.json())
      .then(json => dispatch({
        type: 'ADD_SUGGESTIONS',
        suggestions: json.items
      }));
  }
}
{% endhighlight %}

## Async Action Creators

If actions are supposed to be just simple objects, then why is `addSuggestions`
returning a function? This is an example of an **Async Action Creator**. Async
action creators return functions that take one parameter, the Redux dispatch

[redux]: http://gaearon.github.io/redux/index.html
[the-gist]: http://gaearon.github.io/redux/index.html#the-gist