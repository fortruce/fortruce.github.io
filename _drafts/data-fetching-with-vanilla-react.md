---
title: Data Fetching with Vanilla React
---

React provides a great way to split your applications
view into multiple reusable components, but it does not
provide a particular structure to fetch data for your
application. This is where many people run into problems
when getting started with React, and this series is meant
to introduce several different methods for retrieving data
in your React components.

## Approaches

There are several approaches to fetching data in React applications that will
be covered in this series:

* [Use vanilla React and the Component lifecycle][this-post]
* [Use a Flux based architecture][flux-post]
* [Use Relay and GraphQL][relay-post]

This post will focus on how to fetch data using just vanilla React and the
component lifecycle.

## Vanilla React

We will be building a simple component that tries to autocomplete an input
box by querying Github's search API for usernames.

The `Autocomplete` component will accept a `suggestions` function through
props. This function will be called by `Autocomplete` whenever the input
changes. In this way, the component can be easily reused by supplying a
different suggestions function.

{% highlight js %}
import React, { PropTypes } from 'react';

class Autocomplete extends React.Component {
  static propTypes = {
    suggestions: PropTypes.func.isRequired
  }

  constructor(props) {
    super(props);
    this.state = { search: '', suggestions: [] };
  }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.search !== this.state.search) {
      this.props.suggestions(this.state.search, suggestions => {
        this.setState({ suggestions });
      });
    }
  }

  render() {
    const { suggestions } = this.props;
    const suggestionListItems = this.state.suggestions.map((s, i) => (
        <li key={i}>{ s }</li>
      )
    );
    const suggestionList = suggestionListItems.length ? <ul>{ suggestionListItems }</ul> : null;

    return (
      <div>
        <input
          type="text"
          ref="search"
          value={this.state.search}
          onChange={() => this.setState({
            search: React.findDOMNode(this.refs.search).value
          })} />
        { suggestionList }
      </div>
    );
  }
}
{% endhighlight %}

Whenever the input is updated, the `componentDidUpdate` lifecycle function will
fire off a call to the `suggestions` function. The `suggestions` function expects
the search string to give suggestions for, and a callback to provide the list
of suggestions. Here, the callback takes the list of suggestions and updates
the component's state with the list which will then render under the input.

So, to fetch data from Github's search API, implement the
`suggestions` function that is provided to the `Autocomplete` component to query the API.

{% highlight js %}
const GITHUB_API = 'https://api.github.com';

function searchUsers(user, cb) {
  fetch(`${GITHUB_API}/search/users?q=${user}`)
    .then(res => res.json())
    .then(json => cb(json.items));
}
{% endhighlight %}

Then, supply the `searchUsers` function to fulfill `Autocomplete`
suggestions:

{% highlight js %}
React.render(
  <Autocomplete suggestions={ searchUsers } />,
  document.getElementById('container')
);
{% endhighlight %}

Thus, each time the user types into the input field `this.state.search` is updated
triggering a component update. This update triggers the React `componentDidUpdate`
lifecycle, which, in turn, calls the provided suggestions function to get
suggestions for the new search term. Finally, the callback provided to the `suggestions`
function is called with the results of an ajax call to Github's search API, and
the results are stored in `this.state.suggestions` triggering another component
update.

## Conclusion

This is just one example of using the component lifecycle functions to fetch
data from an asynchronous source (an external API in this case). Other lifecycle
functions can be used to similar effect. For example, making an ajax request
on `componentWillMount` to retrieve data to display in a component.

This approach is fairly simple, but it does not scale well. Placing fetching
logic inside your components makes them harder to reuse and resistant to change.
The following two articles will describe better approaches
for managing fetching data in your React applications.

A working example of all of the code outlined in this article can be found
on [codepen][codepen].

[this-post]: #
[flux-post]: /flux-post
[relay-post]: /relay-post
[codepen]: http://codepen.io/fortruce/pen/JdVOvB