---
layout: post
title:      "A React Quiz App for Developers"
date:       2020-12-10 13:31:31 +0000
permalink:  a_react_quiz_app_for_developers
---


React developers are able to create large web applications which use data that can change over time, without reloading the page. Its main goal is to be fast, simple and scalable. 

## Goal: Build a Quiz App with React + Redux + Rails API
This simple web application will help new developers master programming skills and improve their chances of getting a developer job. The frontend is built with React and Redux, the backend is a Rails REST API with PostgreSQL as database tool.

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
* Solution: Use flashcards to learn, use quiz to test, use job search tool to get a developer job

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
* Quiz Progress tracking
* View quiz result after completing the quiz 
* Option to repeat quiz
* Option to Search specific jobs using the job search form

## Backend 

* Create a new Rails app with PostgeSQL database and limit this Rails app to have only API:

```ruby
rails new quiz-game-api --database=postgresql --api
```

* Next, create a model called Test with name attribute:

```ruby
rails g model Test name

```

* Then create a model called Question with question, answer, and test_id attribute:

```ruby
rails g model Question question answer test_id:integer

```

* Create the database and run the migration:

```ruby
rails db:create && rails db:migrate

```

*   Add rack-cors gem to let the request call from cross domain. Add this line into Gemfile:

```ruby

gem 'rack-cors'

```

### Deploying Rails Backend

* Navigate into the directory of your project’s Rails backend: 
```
cd my-react-app
```
* Sign into Heroku:
```
heroku login
```
* Create Heroku project
 ```
heroku create new-frontend-app
```
* Initialize heroku git remote

```
heroku git remote -a  my-react-app
```

* Deploy your Rails API on Heroku:

```
git push heroku master
```

* [Rails backend live demo](http://online-quiz-api.herokuapp.com/api/v1/tests)
* [Rails backend code on Github](https://github.com/yuanxizhang/quiz-game-api)



## Frontend

Before building a React app, install create-react-app:
```
npm install -g create-react-app
```
Create a new React project:
```
npx create-react-app new-react-frontend

### Set Up the Store and Reducer and Action Creator

```javascript
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
### Set up the reducers

```javascript
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

The frondend React app has 3 container components:

1. FlashcardsCaintainer
2. QuizGamesContainer
3. JobsContainer

*  The FlashcardsContainer and GamesContainer uses Hooks as its state management tool.
*  The JobsContainer uses Redux as its state management tool. 


### Dispatch the fetchJobs Action in ComponentDidMount function

### Add a job search form in JobsContainer for finding jobs by job title and location.

### Install Axios for making an asynchronous fetch request.

Install axios with command: 

```
npm install axios.
```


