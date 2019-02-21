---
layout: post
title:      "Building a sinatra app called Book Club Literature Log"
date:       2019-02-21 05:23:49 +0000
permalink:  building_a_sinatra_app_called_book_club_literature_log
---

We are going to build a book club literature log app for book lovers!

Step 1: Install the Sinatra Ruby Gem, next install the 'shotgun' gem, which we'll use later to serve our web app. Enter the following into the Terminal:

```ruby
gem install sinatra
gem install shotgun
```

Create an empty project, add Gemfile, config.ru. Create and use a Rakefile to run ActiveRecord migrations.

```
cd ~/Desktop
mkdir my-sinatra-project
cd my-sinatra-project
git init
```

In the Gemfile, use sqlite3 for development, use Postgresql for production.

```ruby
gem "pg", :group => :production
gem "sqlite3-ruby", :group => :development
```

In the environment.rb file, setup two separate database configurations for development and production:

```ruby
configure :development do
  ENV['SINATRA_ENV'] ||= "development"
  ActiveRecord::Base.establish_connection(
    :adapter => "sqlite3",
    :database => "db/neighborhood#{ENV['SINATRA_ENV']}.sqlite"
end

configure :production do
  db = URI.parse(ENV['DATABASE_URL'] || 'postgres://localhost/mydb')
  ActiveRecord::Base.establish_connection(
    :adapter => db.scheme == 'postgres' ? 'postgresql' : db.scheme,
    :host     => db.host,
    :username => db.user,
    :password => db.password,
    :database => db.path[1..-1],
    :encoding => 'utf8'
  )
end
```

Step 2: Setup a database in the Sinatra application

Use pen and paper or free online tools to create an Entity Relation Diagram to show the has-many and belongs-to relationship between our models.

Create five tables: users, logs, book_clubs, meetings, and user_meetings.

```ruby
rake db:create_migration NAME=create_meetings
```

Step 3. Build the models:

```ruby
class Meeting < ActiveRecord::Base
  belongs_to :book_club
  has_many :user_meetings
  has_many :users, through: :user_meetings

  validates :topic, presence: true
  validates :date_and_time, presence: true
  validates :location, presence: true
end
```

Step 4:  Use RESTful routes in the controller to create a new meeting, show all meetings, update a meeting, and delete a meeting. Build the routes for user authentication, and routes for create, read, and update records in meetings controller and book clubs controller.

```ruby
get '/meetings' do
  @meetings = Meeting.all
  erb :'/meetings/meetings'
end
```

Step 5: Create views for each model to desplay the records and forms, start shotgun server to test the Sinatra app and make sure the app is working.

Step 6. Install Heroku. Create an account at heroku.com. Deploy the app on Heroku.

```
brew tap heroku/brew && brew install heroku
heroku login

Enter the email, and password for your Heroku account. 
 
```
heroku create my-sinatra-app
```

Run `bundle install`

```
git add .
git commit -m "ready to deploy my-sinatra-app"
git push heroku master
```

Links:
-[Sinatra the book](https://sinatra-org-book.herokuapp.com/)
-[Deploy your app on Heroku using git](https://devcenter.heroku.com/articles/git)
-[Preparing a Codebase for Heroku Deployment](https://devcenter.heroku.com/articles/preparing-a-codebase-for-heroku-deployment)

