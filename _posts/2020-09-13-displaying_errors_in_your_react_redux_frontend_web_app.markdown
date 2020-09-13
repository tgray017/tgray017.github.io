---
layout: post
title:      "Displaying Errors in your React/Redux Frontend Web App"
date:       2020-09-13 15:37:11 +0000
permalink:  displaying_errors_in_your_react_redux_frontend_web_app
---


Every coder knows there are millions of ways to solve a problem. Turns out, that's just as true when it comes to figuring out what to do with the problems themselves--meaning errors. There are many different ways to display errors to users in your frontend React application, but I'm going to walk you through the way it worked for me.

First, understand how errors are being returned from the API you're fetching from. In my situation, errors can come in either the form of a string or an array in the `errors` attribute of the JSON response. The first step for me was to create a function to normalize this, which I called `alertify` and placed in src/Utils.js. The [Utils.js file is a standard location](https://stackoverflow.com/questions/32888728/correct-way-to-share-functions-between-components-in-react) to place functions that could be used by multiple components, so it makes sense to put this here since I may have to set an alert when the user logs in with the wrong password, signs up without invalid credentials, etc...:
```
/* src/Utils.js */

export const alertify = (message) => {
  let messages = []
  if (typeof message === "string") {
    messages.push(message)
  } else {
    message.forEach(m => messages.push(m))
  }
  return messages
}
```
This ensures that whichever format the `message` input takes on, it will ultimately be transformed into an array of messages.

Secondly, since we're in a React app, and in this situation using [Redux](https://redux.js.org/), it would make sense to create a separate reducer responsible for storing the alerts. Add your `alertReducer` to your `reducers` directory:
```
/* src/reducers/alertReducer.js */

export default function alertReducer(state = {
  showAlert: false,
}, action) {
  switch (action.type) {
    default:
      return state
  }
}
```
Don't worry, we're not finished with this yet. So far we're just setting it up to return an initial state of `showAlert: false`--we don't want to bombard our users with an alert the second they load up the app. Then, add this reducer to your `combineReducers` object, usually in your `index.js`:
```
/* src/index.js */

import React from 'react'
import { createStore, applyMiddleware, compose, combineReducers } from 'redux'
import alertReducer from './reducers/alertReducer'
/* ... more imports */

const reducers = {
  /* ... more reducers */
  alerts: alertReducer
}
const reducer = combineReducers(reducers)
...
```

Next, we're going to create an action creator which, when called, will be responsible for dispatching the alert. Note that this does not return a POJO as is standard for action creators, but instead returns a function, which is possible when using the [redux-thunk](https://github.com/reduxjs/redux-thunk) middleware:
```
/* src/actions/setAlert.js */

export const setAlert = (status, message) => {
  return (dispatch) => {
    dispatch({type: 'SET_ALERT', payload: {
      status: status,
      message: message
    }})
  }
}
```
This action expects to be passed a status (e.g "error" or "success") and a message in the form of an array, like ["Password incorrect"], or ["User already exists", "Password confirmation does not match Password"]. It makes use of the Redux store's `dispatch` function, which allows us to communicate with the store asynchronously. Let's go back to the reducer and add in logic to handle this new action:
```
/* src/reducers/alertReducer.js */

export default function alertReducer(state = {
  showAlert: false,
}, action) {
  switch (action.type) {
    case 'SET_ALERT':
      return {
        ...state,
        showAlert: true,
        status: action.payload.status,
        message: action.payload.message
      }

    default:
      return state
  }
}
```
Here we're telling the Redux store to set `showAlert` to true and to pass in the status and message from the action creator. But we still haven't gotten to actually displaying the alert to the user. Let's create a new container component, AlertContainer, that will be responsible for communicating with the store:
```
import React, { Component } from 'react'
import AlertComponent from '../components/AlertComponent'
import { connect } from 'react-redux'

class AlertContainer extends Component {

render() {
  if (this.props.showAlert) {
    return (
      <AlertComponent
        status={this.props.status}
        message={this.props.message}
        hideAlert={this.props.hideAlert}
      />
    )
  } else {
    return null
  }
}

const mapStateToProps = state => {
  return {
    showAlert: state.alerts.showAlert,
    status: state.alerts.status,
    message: state.alerts.message
  }
}

export default connect(mapStateToProps)(AlertContainer)

```
In this container, we're using mapStateToProps to access showAlert, status and message from the store. We're then passing in those props to a new presentational component, AlertComponent, *only* if showAlert is truthy. If showAlert is falsy, AlertContainer will render null, which effectively means it won't render at all.

I know what you're thinking--are we ever going to actually display these alerts to the user?? Well fear not! That's the next step. Let's create the AlertComponent:
```
/* src/components/AlertComponent.js */

import React, { Component } from 'react'
import Alert from 'react-bootstrap/Alert'

export default class AlertComponent extends Component {

  renderErrorMessage = () => {
    const listItems = this.props.message.map((message) => <li>{message}</li>)
    return (
      <ul className="mb-0">
        {listItems}
      </ul>
    )
  }

  render() {
    let variant = this.props.status === 'success' ? 'success' : 'danger'
    return (
      <Alert
        variant={variant}
      >
        {this.renderErrorMessage()}
      </Alert>
    )
  }
}
```
There's a bit going on here--let's break it down. First, we're importing the [Alert component](https://react-bootstrap.github.io/components/alerts/) from react-bootstrap, which will give us some styling and flexibility for showing the alert. Then, in the `render` method, we're rendering that component with either "success" or "danger" variants, depending on the status passed in from the parent component (AlertContainer). Within the Alert component, we're calling `renderErrorMessage`, which takes the alert message from props and turns it into a bulleted list.

The good thing about setting up our components this way is that we're not constricted to only showing error messages, but we can show success messages too, thereby leaving this functionality extensible. For example, if this were a music listening app, we could notify users with a success message whenever a song was successfully added to their library.

At this point, however, there's a major bit of functionality missing here. As is, this alert will stay displayed in the browser indefinitely, which would likely get annoying for our users. Let's change that so that the alert goes away after a certain amount of time, and so the user can dismiss it themselves if they want to.

First, we'll create a simple action creator, called hideAlert:
```
/* src/actions/hideAlert.js */

export const hideAlert = () => {
  return (dispatch) => {
    dispatch({type: 'HIDE_ALERT'})
  }
}
```
Now, we can go back to the alertReducer and handle this new action:
```
/* src/reducers/alertReducer.js */

export default function alertReducer(state = {
  showAlert: false,
}, action) {
  switch (action.type) {
    case 'SET_ALERT':
      return {
        ...state,
        showAlert: true,
        status: action.payload.status,
        message: action.payload.message
      }

    case 'HIDE_ALERT':
      return {
        ...state,
        showAlert: false
      }

    default:
      return state
  }
}
```
All we're doing here is toggling `showAlert` to false in the Redux store. Then, we'll make a few adjustments to our AlertContainer:
```
/* src/containers/AlertContainer.js */

import React, { Component } from 'react'
import { hideAlert } from '../actions/hideAlert'
import AlertComponent from '../components/AlertComponent'
import { connect } from 'react-redux'

class AlertContainer extends Component {

  render() {
    if (this.props.showAlert) {
      setTimeout(() => {
        this.props.hideAlert()
      }, 10000)

      return (
        <AlertComponent
          status={this.props.status}
          message={this.props.message}
          hideAlert={this.props.hideAlert}
        />
      )
    } else {
      return null
    }
  }
}

const mapStateToProps = state => {
  return {
    showAlert: state.alerts.showAlert,
    status: state.alerts.status,
    message: state.alerts.message
  }
}

const mapDispatchToProps = dispatch => {
  return {
    hideAlert: () => dispatch(hideAlert())
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(AlertContainer)
```
You'll notice we're now mapping dispatch to props so we can pass in hideAlert to the AlertComponent. We also added in a setTimeout to the render method, in which it will call hideAlert after 10 seconds. This will set showAlert to false in the store, which will trigger a re-render since there's been a change in props, at which point showAlert will be false and AlertContainer will not render. So, why do we need to pass in hideAlert to the AlertComponent if we're already calling it from AlertContainer anyways? Well, if a user wants to dismiss the alert before 10 seconds is up, they'll have no way to do so unless the AlertComponent itself has access to that action.

Let's go back to the AlertComponent, where we've made a few updates:
```
import React, { Component } from 'react'
import Alert from 'react-bootstrap/Alert'

export default class AlertComponent extends Component {

  handleClose = () => {
    this.props.hideAlert()
  }

  renderErrorMessage = () => {
    const listItems = this.props.message.map((message) => <li>{message}</li>)
    return (
      <ul className="mb-0">
        {listItems}
      </ul>
    )
  }

  render() {
    let variant = this.props.status === 'success' ? 'success' : 'danger'
    return (
      <Alert
        variant={variant}
        onClose={this.handleClose}
        dismissible
      >
        {this.renderErrorMessage()}
      </Alert>
    )
  }
}
```
With this update, we added the "dismissable" flag to Alert, which will display an "x" next to the alert that the user can click to dismiss it. We're also calling `handleClose` whenever that dismiss flag is clicked, which will in turn call `hideAlert`.

Now, whenever we're pinging our API and deciding what to do with the response, we simply do so like this:
```
/* src/actions/someAction.js */

import { setAlert } from './setAlert'
import { alertify } from '../Utils.js'

return (dispatch) => {
  fetch('http://someapi.com')
  .then(response => response.json())
  .then(obj => {
    if (obj.errors) {
      dispatch(setAlert('error', alertify(obj.errors)))
    } else {
      doSomethingWith(obj)
    }
  })
}
```
This is assuming that our JSON response comes with an `errors` key when an error occurs on the server side. If you wanted, you could also add a `catch` to handle and display network errors to your users.

And that's it! We've abstracted the functionality of displaying alerts into separate components, so that wherever in our app an error (or success) occurs, we'll be able to notify our users easily and consistently. I hope this was informative and helps you with your next React Redux app!
