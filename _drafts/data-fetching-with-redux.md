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

## 

[redux]: http://gaearon.github.io/redux/index.html
[the-gist]: http://gaearon.github.io/redux/index.html#the-gist