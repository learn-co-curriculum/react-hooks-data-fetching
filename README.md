# Asynchronous React

## Problem Statement

We've seen that React components come with some neat-o bells and whistles. They
can be nested within each other. They can pass information and logic between
them with props and they can keep track of their own information in state.

So far though, we've been restricted to displaying information organized by the
React app itself. In this lesson, we're going to go a step further and
incorporate remote data into our React apps. Using fetch requests to APIs, we
can build dynamic, responsive apps around data that is provided to us remotely.

## Objectives

- Introduce the use of `fetch` within components
- Consider some of the best places to include `fetch` in our React app

#### Using `fetch` Within React

For a minute, consider how a site like [Instagram][insta] works. If you've got
an account on Instagram, when you visit the site, you'll see an endless scroll
of photos from people you follow. Everyone sees the same _Instagram_ website,
but the images displayed are unique to the user.

Similarly, consider [IMDb, the Internet Movie Database][imdb]. When you click to
look at a movie's information, the page is always the same. The data, the
images, the reviews, the cast... this information changes.

Both of these websites are built with React. When you go to one of these sites,
React doesn't have the specific movie or image content. If you're on a slow
connection (or [want to mimic one using the Chrome Dev Tools][fake3g]), you can
see what is happening more clearly. _React_ shows up first and renders
_something_. Sometimes it is just the background or the skeleton of a website, or
maybe navigation and CSS. On Instagram, a photo 'card' might appear but without
an image or username attached.

React is _mounting_ its basic components _first_. Once these are mounted, remote
data is then requested. When that data has been received, React runs through an
update of the necessary components and fills in the info it received. Text
content will appear, user information, etc... This first set of data is likely
just a JSON object specific to the user or content requested. This object might
contain image URLs, so right after the component update, images will be able
to load.

So, since the data is being requested _after_ React has mounted its components,
is there a component lifecycle method that might be useful here?

Why yes there is! `componentDidMount` happens to be a great place for making
fetch requests. By putting a `fetch()` within `componentDidMount`, when the data
is received, we can use `setState` to store the received data. This causes an
update with that remote data stored in state. A very simple implementation of
the App component with `fetch` might look like this:

```js
import React, { Component } from 'react'

class App extends Component {

  state = {
    peopleInSpace: []
  }

  render() {
    return (
      <div>
        {this.state.peopleInSpace.map(person => person.name)}
      </div>
    )
  }

  componentDidMount() {
    fetch('http://api.open-notify.org/astros.json')
      .then(response => response.json())
      .then(data => {
        this.setState({
          peopleInSpace: data.people
        })
      })
  }
}

export default App
```

In the code above, once App mounts, a `fetch` is called to an API. Once data is
returned from the API, the simplest way to store some or all of it is to put it in
state.

If you have JSX content reliant on that state information, when `setState` is
called and the component re-renders, the content will appear.

Placing `fetch` in `componentDidMount` is ideal for data that you need
immediately when a user visits your website or uses your app. Since
`componentDidMount` is also commonly used to initialize intervals, it is ideal
to set up any repeating fetch requests here as well.

#### Using `fetch` With Events

We aren't limited to sending fetch requests when a component is mounted. We can
also tie them into events:

```js
handleClick = event => {
  fetch('your API url')
    .then(res => res.json())
    .then(json => this.setState({data: json}))
}

render() {
  return (
    <button onClick={this.handleClick}>Click to Fetch!</button>
  )
}
```

This lets us send requests on demand. Submitting form data would be handled this
way, using a POST request instead of GET.

A slightly more complicated example would be the infinite scroll of sites like
Instagram. An event listener tied to changes in the scroll position of a page
could fire off a `handleScroll` method that requests data before a user reaches
the bottom of a page.

#### Using State with POST Requests

One of the beautiful features of state is that we can organize it however we
need. If we were building a form to submit to a server, we can structure state
to work nicely with what the server is expecting in a POST request.

Say we were building a user sign up form. When we send the data, our server is
expecting two values within the body of the POST, `username` and `password`.

Setting up a React controlled form, we can structure our state in the same way:

```js
state = {
  username: "",
  password: ""
}

//since the id values are the same as the keys in state, we can write an abstract setState here
handleChange = event => {
  this.setState({
    [event.target.id]: event.target.value
  })
}

render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <input type="text" id="username" value={this.state.username} onChange={this.handleChange}/>
      <input type="text" id="password" value={this.state.password} onChange={this.handleChange}/>
    </form>
  )
}
```

Then, when setting up the fetch request, we can just pass the entire state within the
body, as there are no other values:

```js
handleSubmit = event => {
  event.preventDefault()
  fetch('the server URL', {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(this.state)
  })
}
```

Notice how we're not bothering to worry about `event.target` when posting the
data. Since the form is controlled, state contains the most up-to-date form
data, and it is already in the right format!

## Conclusion

There are no hard and fast rules for how to include fetch requests, and a lot of
structure will depend on the data you're working with. As a general practice for
writing simpler component code, include `fetch` calls in the same component as
your top level state. You can always pass down methods as props that contain
`fetch`.

## Resources

- [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)

[insta]: https://www.instagram.com/
[imdb]: https://www.imdb.com/
[fake3g]: https://developers.google.com/web/tools/chrome-devtools/network-performance/network-conditions
