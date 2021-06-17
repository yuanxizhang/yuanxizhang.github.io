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
[Quiz Game live demo](https://quiz-box.netlify.app/)

## Table of contents
* [Problem](#Problem)
* [Component](#Components)
* [Technologies](#Technologies)
* [Features](#features)
* [Frontend](#Frontend)
* [Backend](#backend)

## Problem

* Problem: new developers need simple tools to learn, to test their knowledge level, and to find a job
* Solution: use flashcards to learn, use quiz for assessment, use problem and solutions to prepare for coding challenges during the interview, use job search tool to get a developer job,

## Components
<img src="https://github.com/yuanxizhang/quiz-game-frontend/blob/master/public/img/diagram.png"
     alt="React Components"
     style="float: left; margin-right: 10px;" />

*  The App component is a container with React Router. It has a navbar that links to routes paths.
*  The FlashcardsContainer, QuizGameContainer and ProblemsContainer calls DataService functions which use axios to make HTTP requests to and receive responses from Rails API.
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
* Option to create new coding challenges and new solutions
* Option to search developer jobs through the Github job listing

## Backend 

First, create a new Rails API app with PostgeSQL database:

```
rails new quiz-game-api --database=postgresql --api

```

Next, create a model called Test with a name attribute:

```
rails g model Test name

```

Then create a model called Question with question, answer, and explain attributes:

```
rails g model Question question answer explain test_id:integer

```
 create a model called Option with an item attribute:

```
rails g model Option item question_id:integer

```
The relationship is test has many questions, the question has many options.
```
class Test < ApplicationRecord
    has_many :questions, dependent: :destroy
    validates :name, presence: true
    accepts_nested_attributes_for :questions
end

class Question < ApplicationRecord
    belongs_to :test
    has_many :options, dependent: :destroy
    validates :question, presence: true
    validates :answer, presence: true
    accepts_nested_attributes_for :options
end

class Option < ApplicationRecord
    belongs_to :question
    validates :item, presence: true
end
```

Create the database and run the migration:

```
rails db:create && rails db:migrate

```
Build the controllers for all of the models.

Add rack-cors gem to let the request call from cross domain. Add this line into Gemfile:

```ruby
gem 'rack-cors'

```

Update cors.rb fiel in config/initializer directory to allow any origons:!

```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

### Deploying Rails Backend to Heroku

Navigate into the directory of your project’s Rails backend: 

```
cd my-react-app
```

Sign into Heroku:

```
heroku login
```

Create Heroku project:

 ```
heroku create new-frontend-app
```

Initialize heroku git remote:

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
npx create-react-app my-quiz-game
```

### Set Up the Store and Reducer and Action Creator

There are three building blocks that Redux is made of:

1.  Action:  Actions are simple JavaScript objects that describe the course of action concerning what happened to the application’s state without specifying how the application state changed.
2. Reducer:  Reducers are pure functions that present how the application state changes. As soon as the action dispatches to the store, the Reducer starts updating the course of action being passed.
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

1. Step one, we build the index.js file, it creates a global store and calls the App component: <App />.
2. Step two, we build the App.js file, it sets up the routes, and provide the links to the container components. Since App.js is the root component, and every other component is the child component, the state we defined is available to all components.
3.  Step three, we build the container components.

### Set up the Reducers and Action Creator

The reducer function takes two arguments:
*  The previous state of the app
*  The dispatched action that will return the new app state. (previousState, action) => newState


### Build the Container Components

This React app is a bundle of four simple apps:

1. The Flashcard app
2. The Quiz app
3. The Job Ssearch app
4. The Coding Problems and Solutions app

*  The Flashcard app, the Quiz app, and the Problems and Solutions app use Hooks as its state management tool.
*  The Job Search app uses Redux as its state management tool. 

When the container components are ready, we add child components inside the each container components. We want to add a JobsSearch component in JobsContainer to filter jobs by job title and location. 

## Connect your React App to a REST API

To fetch data from a REST API, you perform an asynchronous fetch request to a REST API which will return the requested information.

I used axios library to get quiz data from the Rails API I built, I used javascript fetch request to get job listings from Github Jobs API.

### Install axios for making an asynchronous fetch request.

Install axios with command: 

```
npm install axios.
```

### Fetch Data into a React Component

If you use Redux to manage the state of your application:

*  Add the `componentDidMount` lifecycle method to a class component.
*  Import axios or use javascript fetch function
*  Add the data fetching function 'axios.get()' to `componentDidMount` method to retrieve the data and store it in the App component’s State.

If you use React Hooks to manage the state:

*  Import `useState`, `useEffect`, `useRef` from React library.
*  Import axios or use javascript fetch function

Declare the variables to hold each part of the state:
``` 
const [loading, setLoading] = useSta
```

Add the axios.get() request to useEffects function to retrieve the data.

### Display the data in your user interface

The last step is passing the State data to the class component through props. 
```
{this.props.loading && <h4>Loading...</h4>}
```

If you use React hooks to manage state in a functional component, you can directly use the state related variables in your component:
```
{loading && <h4>Loading...</h4>}
```

Congratulations! You just connected your React app to the remote REST API.

