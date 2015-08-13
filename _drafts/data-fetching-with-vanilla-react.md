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

There are several approaches to fetching data in React applications that will
be covered in this series:

* [Use vanilla React and the Component lifecycle][this-post]
* [Use a Flux based architecture][flux-post]
* [Use Relay and GraphQL][relay-post]

This post will focus on how to fetch data using just vanilla React and the
component lifecycle.

We will be building a simple application that tries to autocomplete an input
box by querying Github's search API for usernames.

{% highlight js %}
import React from 'react';

class GithubAutocomplete extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      search: ''
    }
  }

  onChange = () => {
    this.setState({
      search: React.findDOMNode(this.refs.search).value
    });
  }

  render() {
    return (
      <div>
        <input
          type="text"
          ref="search"
          value={ this.state.search }
          onChange={ this.onChange } />
      </div>
    );
  }
}
{% endhighlight %}

[this-post]: #
[flux-post]: /flux-post
[relay-post]: /relay-post
[codepen]: http://codepen.io/fortruce/pen/JdVOvB