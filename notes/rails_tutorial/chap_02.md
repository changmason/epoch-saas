# Chapter 2 A toy app
  A high-level overview of Ruby on Rails programming (and web development in general) by rapidly generating an application using scaffold generators

## 2.1 Planning the application
New rails project
```shell
$ cd ~/workspace
$ rails _4.2.0.rc2_ new toy_app
$ cd toy_app/
```

Update `Gemfile`
```ruby
source 'https://rubygems.org'

gem 'rails',        '4.2.0.rc2'
gem 'sass-rails',   '5.0.0.beta1'
gem 'uglifier',     '2.5.3'
gem 'coffee-rails', '4.1.0'
gem 'jquery-rails', '4.0.0.beta2'
gem 'turbolinks',   '2.3.0'
gem 'jbuilder',     '2.2.3'
gem 'sdoc',         '0.4.0', group: :doc

group :development, :test do
  gem 'sqlite3',     '1.3.9'
  gem 'byebug',      '3.4.0'
  gem 'web-console', '2.0.0.beta3'
  gem 'spring',      '1.1.3'
end

group :production do
  gem 'pg',             '0.17.1'
  gem 'rails_12factor', '0.0.2'
end
```

and lock the gems
```shell
$ bundle install --without production
```

Init the repo and commit
```shell
$ git init
$ git add -A
$ git commit -m "Initialize repository"
```

Push to remote
```shell
$ git remote add origin git@bitbucket.org:<username>/toy_app.git
$ git push -u origin --all # pushes up the repo and its refs for the first time
```

Create hello action
```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception

  def hello
    render text: "hello, world!"
  end
end
```

Update root path
```ruby
config/routes.rb
  Rails.application.routes.draw do
  ...
  # You can have the root of your site routed with "root"
  **root 'application#hello'**
  ...
end
```

Commit then push to heroku
```shell
$ git commit -am "Add hello"
$ heroku create
$ git push heroku master
```

### 2.1.1 A toy model for users
The user model/table
[fig 2.2](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_user_model.png "The data model for users")

### 2.1.2 A toy model for microposts
[fig 2.3](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_micropost_model.png, "The data model for microposts")
We’ll see how this `user_id` attribute allows us to succinctly express the notion that a user potentially has many associated microposts.

## 2.2 The Users resource
Use the `scaffold` generator to generate the Users resource
```shell
$ rails generate scaffold User name:string email:string
      invoke  active_record
      create    db/migrate/20140821011110_create_users.rb
      create    app/models/user.rb
      invoke    test_unit
      create      test/models/user_test.rb
      create      test/fixtures/users.yml
      ...
```

Migrate the database
```shell
$ bundle exec rake db:migrate
==  CreateUsers: migrating ====================================================
-- create_table(:users)
   -> 0.0017s
==  CreateUsers: migrated (0.0018s) ===========================================
```

Then start the server
```shell
$ rails server -b $IP -p $PORT    # Use only `rails server` if running locally
```

### 2.2.1 A user tour
Available paths/pages
| URL           | Action | Purpose                     |
|---------------|--------|-----------------------------| 
| /users        | index  | page to list all users      |
| /users/1      | show   | page to show user with id 1 |
| /users/new    | new    | page to make a new user     |
| /users/1/edit | edit   | page to edit user with id 1 |

### 2.2.2 MVC in action
[fig 2.11](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/mvc_detailed.png "A detailed diagram of MVC in Rails")

Summary of the steps
  1. The browser issues a request for the /users URL.
  2. Rails routes /users to the index action in the Users controller.
  3. The index action asks the User model to retrieve all users (User.all).
  4. The User model pulls all the users from the database.
  5. The User model returns the list of users to the controller.
  6. The controller captures the users in the @users variable, which is passed to the index view.
  7. The view uses embedded Ruby to render the page as HTML.
  8. The controller passes the HTML back to the browser.

Remove the `hello` action from `application` controller, and update root route
See users resource
```ruby
# config/routes.rb
Rails.application.routes.draw do
  resources :users
  root 'users#index'
  .
end
```

The `UsersController` overview
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  def index
    ...
  end

  def show
    ...
  end

  def new
    ...
  end

  def edit
    ...
  end

  def create
    ...
  end

  def update
    ...
  end

  def destroy
    ...
  end
end
```

RESTful route mapping about the Users resource

| HTTP request | URL           | Action  | Purpose                     |
|--------------|---------------|---------|-----------------------------|
| GET          | /users        | index   | page to list all users      |
| GET          | /users/1      | show    | page to show user with id 1 |
| GET          | /users/new    | new     | page to make a new user     |
| POST         | /users        | create  | create a new user           |
| GET          | /users/1/edit | edit    | page to edit user with id 1 |
| PATCH        | /users/1      | update  | update user with id 1       |
| DELETE       | /users/1      | destroy | delete user with id 1       |

Pick one route, and take a look at the controller, model and view code.

### 2.2.3 Weaknesses of this Users resource
  - **No data validations.** Our User model accepts data such as blank names and invalid email addresses without complaint.
  - **No authentication.** We have no notion of logging in or out, and no way to prevent any user from performing any operation.
  - **No tests.** This isn’t technically true—the scaffolding includes rudimentary tests—but the generated tests don’t test for data validation, authentication, or any other custom requirements.
  - **No style or layout.** There is no consistent site styling or navigation.
  - **No real understanding.** If you understand the scaffold code, you probably shouldn’t be reading this book.

## 2.3 The Microposts resource

### 2.3.1 A micropost microtour
Use `rails generate` to generate the Microposts resource
```shell
$ rails generate scaffold Micropost content:text user_id:integer
      invoke  active_record
      create    db/migrate/20140821012832_create_microposts.rb
      create    app/models/micropost.rb
      invoke    test_unit
      create      test/models/micropost_test.rb
      create      test/fixtures/microposts.yml
      invoke  resource_route
       route    resources :microposts
       ...
```

Migrate the database again
```shell
$ bundle exec rake db:migrate
==  CreateMicroposts: migrating ===============================================
-- create_table(:microposts)
   -> 0.0023s
==  CreateMicroposts: migrated (0.0026s) ======================================
```

RESTful route mapping about the Microposts resource

| HTTP request | URL                | Action  | Purpose                          |
|--------------|--------------------|---------|----------------------------------|
| GET          | /microposts        | index   | page to list all microposts      |
| GET          | /microposts/1      | show    | page to show micropost with id 1 |
| GET          | /microposts/new    | new     | page to make a new micropost     |
| POST         | /microposts        | create  | create a new micropost           |
| GET          | /microposts/1/edit | edit    | page to edit micropost with id 1 |
| PATCH        | /microposts/1      | update  | update micropost with id 1       |
| DELETE       | /microposts/1      | destroy | delete micropost with id 1       |

The `MicropostsController` overview
```ruby
# app/controllers/microposts_controller.rb
class MicropostsController < ApplicationController
  def index
    ...
  end

  def show
    ...
  end

  def new
    ...
  end

  def edit
    ...
  end

  def create
    ...
  end

  def update
    ...
  end

  def destroy
    ...
  end
end
```

### 2.3.2 Putting the micro in microposts
Limit the length of the post content
```ruby
# app/models/micropost.rb
class Micropost < ActiveRecord::Base
  **validates :content, length: { maximum: 140 }**
end
```

[fig 2.14](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_length_error_3rd_edition.png "Error messages for a failed micropost creation.")

### 2.3.3 A user has_many microposts
Associate the User and Micropost models
[fig 2.15](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/micropost_user_association.png "The association between microposts and users")

Declare a `has_many` association for User
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  **has_many :microposts**
end
```

Declare a `belongs_to` association for Micropost
```ruby
# app/models/micropost.rb
class Micropost < ActiveRecord::Base
  **belongs_to :user**
  validates :content, length: { maximum: 140 }
end
```

Experiment in console
```shell
$ rails console
>> first_user = User.first
=> #<User id: 1, name: "Michael Hartl", email: "michael@example.org",
created_at: "2014-07-21 02:01:31", updated_at: "2014-07-21 02:01:31">
>> first_user.microposts
=> [#<Micropost id: 1, content: "First micropost!", user_id: 1, created_at:
"2014-07-21 02:37:37", updated_at: "2014-07-21 02:37:37">, #<Micropost id: 2,
content: "Second micropost", user_id: 1, created_at: "2014-07-21 02:38:54",
updated_at: "2014-07-21 02:38:54">]
>> micropost = first_user.microposts.first    # Micropost.first would also work.
=> #<Micropost id: 1, content: "First micropost!", user_id: 1, created_at:
"2014-07-21 02:37:37", updated_at: "2014-07-21 02:37:37">
>> micropost.user
=> #<User id: 1, name: "Michael Hartl", email: "michael@example.org",
created_at: "2014-07-21 02:01:31", updated_at: "2014-07-21 02:01:31">
>> exit
```

### 2.3.4 Inheritance hierarchies
Inheritance relationship of the models
```ruby
# app/models/user.rb
class User < ActiveRecord::Base
  ...
end
```

```ruby
# app/models/micropost.rb
class Micropost < ActiveRecord::Base
  ...
end
```

[fig 2.16](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_model_inheritance.png "The inheritance hierarchy for the User and Micropost models")

Inheritance relationship of controllers
```ruby
# app/controllers/users_controller.rb
class UsersController < ApplicationController
  ...
end
```

```ruby
# app/controllers/microposts_controller.rb
class MicropostsController < ApplicationController
  ...
end
```

```ruby
# app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  ...
end
```

[fig 2.17](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/demo_controller_inheritance.png "The inheritance hierarchy for the Users and Microposts controllers")

### 2.3.5 Deploying the toy app
Commit and backup to remote first
```shell
$ git status
$ git add -A
$ git commit -m "Finish toy app"
$ git push
```

Deploy to heroku
```shell
$ git push heroku
```

and migrate the database on heroku
```shell
$ heroku run rake db:migrate
```

## 2.4 Conclusion
**Strengths**
  - High-level overview of Rails
  - Introduction to MVC
  - First taste of the REST architecture
  - Beginning data modeling
  - A live, database-backed web application in production

**Weaknesses**
  - No custom layout or styling
  - No static pages (such as “Home” or “About”)
  - No user passwords
  - No user images
  - No logging in
  - No security
  - No automatic user/micropost association
  - No notion of “following” or “followed”
  - No micropost feed
  - No meaningful tests
  - No real understanding

### 2.4.1 What we learned in this chapter
  - Scaffolding automatically creates code to model data and interact with it through the web.
  - Scaffolding is good for getting started quickly but is bad for understanding.
  - Rails uses the Model-View-Controller (MVC) pattern for structuring web applications.
  - As interpreted by Rails, the REST architecture includes a standard set of URLs and controller actions for interacting with data models.
  - Rails supports data validations to place constraints on the values of data model attributes.
  - Rails comes with built-in functions for defining associations between different data models.
  - We can interact with Rails applications at the command line using the Rails console.

## 2.5 Exercises
Please refer to the book!
