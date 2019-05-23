---
layout: post
title:      "Build a Booking App with Rails 5.2"
date:       2019-05-23 00:15:22 +0000
permalink:  build_a_booking_app_with_rails_5_2
---

Want to build a new app? Let's build a Rails app that gives users the ability to book lessons. After reading this blog post, you will know the steps to build a simple Rails app. 

### Objective 
We want to allow users to browse the lessons that are available, check out the reviews and average ratings for each instructor. After the user has logged in, the user can book a lesson for a family member or himself/herself, and post a review of an instructor.

Here is the user interface of the finished [Booking App](https://book-lessons.herokuapp.com) that delivers the features we want.

### Tech Stacks we will use for this project:
#### Back-end: 
* Ruby 2.6
* Rails 5.2
* Database: PostgreSQL 9.6

#### Front-end:
* HTML, CSS
* Bootstrap 


### Create a New Rails Application
#### Install Rails and Postgres
Rails requires Ruby version 2.2.2 or later.
```
brew install ruby
ruby -v
```
Next we can install Rails and Postgres.
```ruby
gem install rails
gem install pg
```

#### Create a role/user for Postgres
Please check [PostgreSQL docs](https://www.postgresql.org/docs/8.1/sql-createrole.html) for detailed instruction.

```ruby
psql
CREATE ROLE booking-app WITH CREATEDB LOGIN PASSWORD 'secretpw';
exit
```

#### Create your new Rails app
By default, Rails application uses a SQLite database for data storage. For this project, we would like to use PostgreSQL for data storage.

```ruby
rails new booking-app -d postgresql
```

or
```ruby
rails new booking-app --database=postgresql
```

If you created your new Rails app without specifying a database type, you can update your Gemfile: 
* Add PostgreSQL gem:
```ruby
gem 'pg'
```

* Remove SQLite gem:
```ruby
gem 'sqlite3' 
```

#### Configure the PostgreSQL Database

You want to update the database configuration file: config/database.yml:

```ruby
default: &default
  adapter: postgresql
  encoding: unicode
  host: localhost
  url: postgresql://localhost/booking_development?pool=5
  user: booking
  password: secretpw
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: booking_development

test:
  <<: *default
  database: booking_test

production:
  <<: *default
  database: booking_production
  url: ENV['DATABASE_URL'] 
  username: booking-app
  password: ENV['DATABASE_PASSWORD']
```
Run `bundle install' after you updated your Gemfile. Next you can use a rake command to create your database:
```ruby
bundle install
rake db:create
```

### The MVC Architecture
#### Set up Models and Migration tables
A model represents the information (data) of the application and the rules to manipulate that data.

![ER Diagram](https://github.com/yuanxizhang/book-lessons/blob/master/app/assets/images/ER_diagram.png?raw=true)

We need to build these models for our project: User, Session, Instructor, Lessons, Skill, Section, Booking, Review.

We have three different pairs of many-to-many relationships, so we have three joint tables: Lessons, Bookings, Reviews.

* Lessons table is a joint table between instructors and skills table.
     - Each instructor can teach many skills, each skill can be taught by many instructors. 
     - The lesson table stores when and where the lesson will be taught, who is the instructor, what skill is taught.
        
* Bookings table is a joint table between users and lessons table.
     - The user can book the same lesson for different studemts in the user's family.
     - The user can submit the student's name and phone number with each booking.
			 
* Reviews table is a joint table between users and instructors table. 
     - Each user can review many instructors, and each instructor can be reviewed by many users. 
     - The Instructor model calculates the total review count and average rating for each instructor.
     - The reviews table stores the reviews and ratings data. 

Let's create the Lesson model and Booking model:
```
rails generate model Lesson
rails generate model Booking
```

The Lesson model has an integer field that tracks how many seats are left.

```ruby
class Lesson < ApplicationRecord
  belongs_to :skill
  belongs_to :instructor
  has_many :bookings
  has_many :users, through: :bookings

  validates :title, presence: true
  scope :available, -> { where(available: true) }
end

```

The Booking model is responsible to update the number of seats left in each lesson, and to set the "available" field to false if there is no more seats left.

```ruby
class Booking < ApplicationRecord
  belongs_to :lesson
  belongs_to :user

  validates :lesson_id, uniqueness: { scope: :user_id }

  def take_lesson
    if self.lesson.seats < 1
        "Sorry. This lesson is sold out!"
    else
        update_lesson
        "Thanks for booking the #{self.lesson.title} lesson!"
    end
  end

  def update_lesson
    if self.lesson.seats > 1
      self.lesson.seats = self.lesson.seats - 1
      self.lesson.save
    else
      self.lesson.seats = 0
      self.lesson.available = false
      self.lesson.save
    end
  end
end
```

We also need to create the migration tables that store the data for all of the models. then we can use a rake command to run the migration:
```
rake db:migrate
```

#### Set up Views
Views represent the user interface of your application. In Rails, views are often HTML files with embedded Ruby code that performs tasks related solely to the presentation of the data. 

Add the Bootstrap CDN links  in app/views/layouts/application.html.erb.

Please check [Bootstrap docs](https://getbootstrap.com/docs/4.3/getting-started/introduction/) for instructions on the latest Bootstrap CDN links needed for a starter template.

Anyone can see the list of all lessons, can view each individual lesson, but only logged in user can book a lesson, write a review and rate an instructor. Only admin user can create new lessons, edit lessons, and delete lessons.

In view, we check if the user is an admin user with this piece of code:
<% if current_user && current_user.admin %>

Let's create an admin dashboard:

First add 'rails_admin' gem to your gem file.
```
gem 'rails_admin'
```
Then type this command in your terminal, hit enter when it prompts you a question.
```
rails generate rails_admin:install
```
This command will create the '/admin' route and rails_admin.rb in config/initializers.

#### Set up Routes and Controllers
In Rails, controllers are responsible for processing the incoming requests from the web browser, interrogating the models for data, and passing that data on to the views for presentation.

Lets include a few nested routes:
```
resources :sections do
    resources :skills, only: [:new, :create]
end

resources :instructors do
    resources :reviews, only: [:index, :new, :create]
end
```

Let's create the lessons controller:
```
rails generate controller Lessons index 
```

### Deploy the app on Heroku
Heroku is a cloud platform that lets companies build, deliver, monitor and scale apps. Heroku is so easy to use that it's a top choice for many development projects.  We will use the [free resources](https://www.heroku.com/free) on Heroku to deploy our app.

#### Download and Install Heroku Command Line Interface (CLI) 
Please check [Heroku CLI docs](https://devcenter.heroku.com/articles/heroku-cli) for detailed instruction on installing Heroku Command Line Interface.
```
brew tap heroku/brew && brew install heroku
```
Check your Heroku installation:
```
heroku --version
```
[Sign up](https://signup.heroku.com/) for a heroku account and login
```
heroku login
```
#### Deploy the Rails app
Please check the detailed instructions from [Heroku Dev Center](https://devcenter.heroku.com/articles/getting-started-with-rails5) on how to deploy Rails app on Heroku.

Make sure you are in the directory that contains your Rails app, then create an app on Heroku:
```
heroku create
heroku apps:rename newname
heroku remote -v
```
Deploy your rails app:
```
git push heroku master
heroku run rake db:migrate
```
Now your Rails app is ready for the world to see!
