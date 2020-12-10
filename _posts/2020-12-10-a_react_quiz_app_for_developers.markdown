---
layout: post
title:      "A React Quiz App for Developers"
date:       2020-12-10 08:31:33 -0500
permalink:  a_react_quiz_app_for_developers
---


React developers are able to create large web applications which use data that can change over time, without reloading the page. Its main goal is to be fast, simple and scalable. 

## Goal: build a quiz app with React + Redux

This simple web application will help new developers master programming skills and improve their chances of getting a developer job. The frontend is built with React and Redux. For the backend, I made a Rails REST API with PostgreSQL database. 

## Demo
[Quiz Game live demo](https://quiz-box.netlify.app/#/flashcards)

## Table of contents
* [Problem](#Problem)
* [Component](#Components)
* [Technologies](#Technologies)
* [Features](#features)
* [Frontend](#Frontend)
* [Backend](#backend)

## Problem

* Problem: New developers need simple tools to learn, to test their knowledge level, and to find a job
* Solution: Use flashcards to learn, use quiz for sel, use job search tool to get a developer job

## Components

*  The App component is a container with React Router. It has a navbar that links to routes paths.
*  The FlashcardsContainer and QuizGameContainer calls TestDataService functions which use axios to make HTTP requests to and receive responses from Rails API.
*  The JobsContainer uses Redux-Thunk middleware to make an asynchronous fetch request to Github Jobs API using action creator functions. 
*  The action creators are connected to the Redux store using the connect function in conjunction with mapStatetoProps and mapDispatchToProps. 

## Technologies

* React 16.13.1
* Redux 4.0.5
* Rails 6.0.3
* PostgreSQL 12.3
* Bootstrap 4.5.2
* axios 0.21.0

## Features

* Option to practice with the flashcards before taking the quiz
* Immediate feedback on the answer choice for each question
* Quiz progress tracking
* View quiz result after completing the test
* Option to repeat quiz
* Option to find developer jobs through the Github job listing

## Backend 

First, create a new Rails app with PostgeSQL database and limit this Rails app to have only API:

```ruby
rails new quiz-game-api --database=postgresql --api
```

Next, create a model called Test with name attribute:

```ruby
rails g model Test name

```

Then create a model called Question with question, answer, and test_id attribute:

```ruby
rails g model Question question answer test_id:integer

```

* Create the database and run the migration:

```ruby
rails db:create && rails db:migrate

```

Add rack-cors gem to let the request call from cross domain. Add this line into Gemfile:

```ruby

gem 'rack-cors'

```

### Deploying Rails Backend

Navigate into the directory of your project’s Rails backend: 
```
cd my-react-app
```

Sign into Heroku:
```
heroku login
```

Create Heroku project
 ```
heroku create new-frontend-app
```

Initialize heroku git remote

```
heroku git remote -a  my-react-app
```

Deploy your Rails API on Heroku:

```
git push heroku master
```

* [Rails backend live demo](http://online-quiz-api.herokuapp.com/api/v1/tests)
* [Rails backend code on Github](https://github.com/yuanxizhang/quiz-game-api)



## Frontend

Before we start building the React app, let's install create-react-app:
```
npm install -g create-react-app
```

Create a new React project:
```
npx create-react-app new-react-frontend

### Set Up the Store and Reducer and Action Creator

There are three building blocks that Redux is made of:

1.  Action:  Actions are simple JavaScript objects that describe the course of action concerning what happened to the application’s state without specifying how the application state changed.
2. Reducer:  Reducers are solid functions that present how the application state changes. As soon as the action dispatches to the store, the Reducer starts updating the course of action being passed.
3.  Store: A Store is an object that holds the functions and state of its application. It is like a hard drive where you can store any data that you want. This object also helps to bring the Reducer and Action together.

Redux is nothing but a JavaScript library used to manage the state of the application and build a user interface. 

Data flow in Redux encompasses all its building blocks along with the connection which combines them together.

When an event has been introduced by the user that cause a change in the original state., the next series of happenings that will take place are as follows:

* First, the event handler function will dispatch an action to the store with the store.dispatch() method.
* The Reducer will get the dispatched action passed on to it by Redux
* The store will then save the new state returned by the Reducer
* As the React components receive the new state from the store, the User interface is updated.

```
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

import { Provider } from 'react-redux'
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import jobsReducer from './reducers/jobsReducer';
import "bootstrap/dist/css/bootstrap.css";
import './index.css';

const store = createStore(jobsReducer, applyMiddleware(thunk))

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);

```

### Set up the reducers.

The reducer function takes two arguments:
*  The previous state of the app
*  The dispatched action that will return the new app state. (previousState, action) => newState

```
const jobsReducer = (state = { jobs: [], loading: false }, action) => {
    switch(action.type) {
      case 'LOADING_JOBS':
        return {
          ...state,
          jobs: [...state.jobs],
          loading: true
        }
      case 'ADD_JOBS':
        return {
          ...state,
          jobs: action.jobs,
          loading: false
        }
      default:
        return state;
    }
  }
   
 export default jobsReducer;
```

### Build the Container Components

This React app has 3 container components:

1. FlashcardsContainer
2. QuizGamesContainer
3. JobsContainer

*  The FlashcardsContainer and GamesContainer uses Hooks as its state management tool.
*  The JobsContainer uses Redux as its state management tool. 

Add a search form in JobsContainer to filter jobs by job title and location.


## Connect your React App to a REST API

To fetch data from a REST API, you perform an asynchronous fetch request to a REST API which will return the requested information.

I used axios library to get quiz data from the Rails API I built, I used javascript fetch request to get job listings from Github Jobs API.

### Install axios for making an asynchronous fetch request.

Install axios with command: 

```
npm install axios.
```

### Fetching Data into a React Component

If you use Redux to manage the state of your application:

*  Add the `componentDidMount` lifecycle method to a class component.
*  Import axios 
*  Add the axios GET request to `componentDidMount` to retrieve the contact data and store it in the App component’s State.

If we use React Hooks to manage the state:

*  Import `useState`, `useEffect`, `useRef` from React library.
*  Import axios or use javascript fetch function
*  Declare the variables to hold each part of the state:

``` 
const [loading, setLoading] = useState(false);
const [flashcards, setFlashcards] = useState([]);
```
*  Add the axios GET request to useEffects function to retrieve the data

The last step is passing the State data to the class component through props: 
```
const jobs = this.props.jobs.map(job => <Job key={job.id} job={job} />); 
```
If you use React hooks to manage state in a functional component, you can directly use the state related variables in your component:

```
<FlashcardList flashcards={flashcards} />
```

Congratulations! You just connected your React app to a remote REST API.

