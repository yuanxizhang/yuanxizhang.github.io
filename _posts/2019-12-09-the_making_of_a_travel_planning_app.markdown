---
layout: post
title:      "The Making of a Travel Planning App"
date:       2019-12-09 16:44:40 -0500
permalink:  the_making_of_a_travel_planning_app
---

Are you passionate about travel around the world doing what you love to do? Do you like hiking, biking, diving, painting, or cooking? You can use this travel planning app to organize a group trip, keep track the best tours offers for your group, add travelers to your group, and keep a list of places each traveler wants to visit.

The travel planning app has a Rails API backend and Javascript frontend.

Here are the tools you might need to build a Rails API:

- Ruby version: 2.6

- Rails version: 6.0

- Database: PostgreSQL

Create the Rails API backend of the application:

Step 1: Build the models for Rails REST API backend. 

In your terminal enter the following command:

rails new travel_plans --database=postgresql --api

Create the models: Traveler, Plan, Offer, Provider

rails g model Traveler name, passion
rails g model Plan place, adventure
rails g model Provider name, website
rails g model Offer tour_name, about, departs, length, price, image, likes

There are two different one-to-many relationships between the models: each traveler has many travel plans, each provider has many tour offers.

Step 2: Create the serializers: 

rails g serializer traveler
rails g serializer plan
rails g serializer provider
rails g serializer offer

Step 3: Create the controllers: 

rails g controller api/v1/Travelers
rails g controller api/v1/Plans
rails g controller api/v1/Providers
rails g controller api/v1/Offers

Things we need to do for the traveler controller:
*  We're rendering all travelers in the form of JSON.
* We're creating a new traveler based on whatever traveler_params we get from our frontend.
* We're setting out traveler_params to permit a parameter named *name* and a parameter named *passion*. These must be included in the body of the POST requests we will be making with JS fetch.

Step 4: Define routes in config/routes.rb:

```
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
    	resources :travelers, only: [:index, :new, :show, :update]
      resources :plans, only: [:index, :new, :show, :update]
      resources :providers, only: [:index, :show, :update]
      resources :offers, only: [:index, :new, show, :update]
    end
  end
end
```


Step 5: create migration tables

rails db:create && rails db:migrate

After the Rails Backend is completed, we can check the four working endpoints, or routes that it exposes to the public. To see all the travelers, for example, we could navigate to http://localhost:3000/api/v1/travelers.

Create the javascript frontend of the application:

In the frontend, we use classes and functions to organize our code into reusable pieces. We translate JSON responses into JavaScript model objects.

We also want to create a class whose only responsibility is to communicate with the Rails API, we can name this class Adapter and save it in the file adapter.js.

In the adapter.js file, we want to use fetch to handle Client-Server Communications. All interactions between the client and the server should be handled asynchronously (AJAX) and use JSON as the communication format. 

Here are some of the instance methods in the Adapter class:

```
get(url) {
    return fetch(url).then(res => res.json()).catch(err => console.log(err));
  }

patch(url, obj) {
    return fetch(url, {
      method: 'PATCH',
      headers: this.headers,
      body: JSON.stringify(obj),
    }).then(res => res.json())
      .catch(err => console.log(err));
}

delete(url, obj) {
    return fetch(url, {
      method: 'DELETE',
      headers: this.headers,
      body: JSON.stringify(obj),
    }).then(res => res.json())
      .catch(err => console.log(err));
}

post(url, obj) {
    return fetch(url, {
      method: 'POST',
      headers: this.headers,
      body: JSON.stringify(obj),
    }).then(res => res.json())
      .catch(err => console.log(err));
}
```

Users can add a new traveler, remove a traveler, add a new travel plans for each traveler, and like a tour offer. 

Enjoy your trips!
