# Chapter 3 Mostly static pages
Begin developing the professional-grade sample application that will serve as our example throughout the rest of this tutorial.
Moreover, having a good test suite will allow us to refactor our code with confidence, changing its form without changing its function.

---

## 3.1 Sample app setup
Generate a new rails sample app
```shell
$ cd ~/workspace
$ rails _4.2.0.rc2_ new sample_app
$ cd sample_app/
```

Update `Gemfile`
```ruby
source 'https://rubygems.org'

gem 'rails',                '4.2.0.rc2'
gem 'sass-rails',           '5.0.0.beta1'
gem 'uglifier',             '2.5.3'
gem 'coffee-rails',         '4.1.0'
gem 'jquery-rails',         '4.0.0.beta2'
gem 'turbolinks',           '2.3.0'
gem 'jbuilder',             '2.2.3'
gem 'sdoc',                 '0.4.0', group: :doc

group :development, :test do
  gem 'sqlite3',     '1.3.9'
  gem 'byebug',      '3.4.0'
  gem 'web-console', '2.0.0.beta3'
  gem 'spring',      '1.1.3'
end

group :test do
  gem 'minitest-reporters', '1.0.5'
  gem 'mini_backtrace',     '0.1.3'
  gem 'guard-minitest',     '2.3.1'
end

group :production do
  gem 'pg',             '0.17.1'
  gem 'rails_12factor', '0.0.2'
end
```

---

and lock the gems
```shell
$ bundle install --without production
```

Ensure bundled the latest gems
```shell
$ bundle update
```

Then commit for the first time
```shell
$ git init
$ git add -A
$ git commit -m "Initialize repository"
```

---

Rename the README file
```shell
$ git mv README.rdoc README.md
```

and update `README.md`
```
# Ruby on Rails Tutorial: sample application

This is the sample application for the
[*Ruby on Rails Tutorial:
Learn Web Development with Rails*](http://www.railstutorial.org/)
by [Michael Hartl](http://www.michaelhartl.com/).
```

---

Commit the changes
```shell
$ git commit -am "Improve the README"
```

Push and backup the repo to remote
```shell
$ git remote add origin git@bitbucket.org:<username>/sample_app.git
$ git push -u origin --all # pushes up the repo and its refs for the first time
```

Add a default action `ApplicationController#hello`, then commit and deploy to heroku
```
$ git commit -am "Add hello"
$ heroku create
$ git push heroku master
```

---

## 3.2 Static pages
Start a new branch to develop static-pages
```shell
$ git checkout master
$ git checkout -b static-pages
```

---

### 3.2.1 Generated static pages
Use `rails generate` to generate the `controller` with specified actions
```shell
$ rails generate controller StaticPages home help
      create  app/controllers/static_pages_controller.rb
       route  get 'static_pages/help'
       route  get 'static_pages/home'
      invoke  erb
      create    app/views/static_pages
      create    app/views/static_pages/home.html.erb
      create    app/views/static_pages/help.html.erb
      ...
```

---

Some commands have handy shortcuts

| Full command     | Shortcut  |
|------------------|-----------|
| $ rails server   | $ rails s |
| $ rails console  | $ rails c |
| $ rails generate | $ rails g |
| $ bundle install | $ bundle  |
| $ rake test      | $ rake    |

---

Commit and push to remote repo's corresponding branch
```shell
$ git status
$ git add -A
$ git commit -m "Add a Static Pages controller"
$ git push -u origin static-pages
```

---
Some commands can be undone
```shell
$ rails generate controller StaticPages home help
$ rails destroy  controller StaticPages home help
```

```shell
$ bundle exec rake db:migrate
$ bundle exec rake db:rollback
```

Start the server and navigate the pages
```shell
$ rails server -b $IP -p $PORT
```

![fig 3.2](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/raw_home_view_3rd_edition.png "The raw home view (/static_pages/home)")

---

See how controller actions and views work together
```ruby
# app/controllers/static_pages_controller.rb
class StaticPagesController < ApplicationController

  def home
  end

  def help
  end
end
```

app/views/static_pages/home.html.erb
```erb
<h1>StaticPages#home</h1>
<p>Find me in app/views/static_pages/home.html.erb</p>
```

app/views/static_pages/help.html.erb
```erb
<h1>StaticPages#help</h1>
<p>Find me in app/views/static_pages/help.html.erb</p>
```

---

### 3.2.2 Custom static pages
app/views/static_pages/home.html.erb
```erb
<h1>Sample App</h1>
<p>
  This is the home page for the
  <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
  sample application.
</p>
```

app/views/static_pages/help.html.erb
```erb
<h1>Help</h1>
<p>
  Get help on the Ruby on Rails Tutorial at the
  <a href="http://www.railstutorial.org/#help">Rails Tutorial help section</a>.
  To get help on this sample app, see the
  <a href="http://www.railstutorial.org/book"><em>Ruby on Rails Tutorial</em>
  book</a>.
</p>
```

---

## 3.3 Getting started with testing
Red, Green and Refactor

### 3.3.1 Our first test
test/controllers/static_pages_controller_test.rb
```ruby
require 'test_helper'

class StaticPagesControllerTest < ActionController::TestCase

  test "should get home" do
    get :home
    assert_response :success
  end

  test "should get help" do
    get :help
    assert_response :success
  end
end
```

Run the test and pass
```shell
$ bundle exec rake test
2 tests, 2 assertions, 0 failures, 0 errors, 0 skips
```

### 3.3.2 Red
test/controllers/static_pages_controller_test.rb
```ruby
require 'test_helper'

class StaticPagesControllerTest < ActionController::TestCase

  ...

  test "should get about" do
    get :about
    assert_response :success
  end
end
```

Run the test but failed
```shell
$ bundle exec rake test
3 tests, 2 assertions, 0 failures, 1 errors, 0 skips
```

---

### 3.3.3 Green
What failed? Got a error
```shell
$ bundle exec rake test
ActionController::UrlGenerationError:
No route matches {:action=>"about", :controller=>"static_pages"}
```

Add missing route
```ruby
# config/routes.rb
Rails.application.routes.draw do
  get 'static_pages/home'
  get 'static_pages/help'
  ***get 'static_pages/about'**
  ...
end
```

---

Got another error
```
$ bundle exec rake test
AbstractController::ActionNotFound:
The action 'about' could not be found for StaticPagesController
```

Add the missing action
```ruby
# app/controllers/static_pages_controller.rb
class StaticPagesController < ApplicationController

  def home
  end

  def help
  end

  def about
  end
end
```

---

Still failed
```shell
$ bundle exec rake test
ActionView::MissingTemplate: Missing template static_pages/about
```

Add the missing view template file
```shell
$ touch app/views/static_pages/about.html.erb
```

app/views/static_pages/about.html.erb
```erb
<h1>About</h1>
<p>
  The <a href="http://www.railstutorial.org/"><em>Ruby on Rails
  Tutorial</em></a> is a
  <a href="http://www.railstutorial.org/book">book</a> and
  <a href="http://screencasts.railstutorial.org/">screencast series</a>
  to teach web development with
  <a href="http://rubyonrails.org/">Ruby on Rails</a>.
  This is the sample application for the tutorial.
</p>
```

---

Everything should pass again
```shell
$ bundle exec rake test
3 tests, 3 assertions, 0 failures, 0 errors, 0 skips
```

![fig 3.5](https://softcover.s3.amazonaws.com/636/ruby_on_rails_tutorial_3rd_edition/images/figures/about_us_3rd_edition.png "The new About page (/static_pages/about)")

---

### 3.3.4 Refactor

---

## 3.4 Slightly dynamic pages
Rename the original layout file
```shell
$ mv app/views/layouts/application.html.erb layout_file
```

We want to have dynamic page title

| Page  | URL                 |  Base title                         | Variable title |
|-------| --------------------|-------------------------------------|----------------|
| Home  | /static_pages/home  | "Ruby on Rails Tutorial Sample App" | "Home"         |
| Help  | /static_pages/help  | "Ruby on Rails Tutorial Sample App" | "Help"         |
| About | /static_pages/about | "Ruby on Rails Tutorial Sample App" | "About"        |

---

### 3.4.1 Testing titles (Red)
Test for the page titles
```ruby
#test/controllers/static_pages_controller_test.rb
require 'test_helper'

class StaticPagesControllerTest < ActionController::TestCase

  test "should get home" do
    get :home
    assert_response :success
    **assert_select "title", "Home | Ruby on Rails Tutorial Sample App"**
  end

  test "should get help" do
    get :help
    assert_response :success
    **assert_select "title", "Help | Ruby on Rails Tutorial Sample App"**
  end

  test "should get about" do
    get :about
    assert_response :success
    **assert_select "title", "About | Ruby on Rails Tutorial Sample App"**
  end
end
```

and got failures
```shell
$ bundle exec rake test
3 tests, 6 assertions, 3 failures, 0 errors, 0 skips
```

---

### 3.4.2 Adding page titles (Green)

Update the 3 view files

---

app/views/static_pages/home.html.erb
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Home | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
      sample application.
    </p>
  </body>
</html>
```

---

app/views/static_pages/help.html.erb
```html
<!DOCTYPE html>
<html>
  <head>
    <title>Help | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Help</h1>
    <p>
      Get help on the Ruby on Rails Tutorial at the
      <a href="http://www.railstutorial.org/#help">Rails Tutorial help
      section</a>.
      To get help on this sample app, see the
      <a href="http://www.railstutorial.org/book"><em>Ruby on Rails
      Tutorial</em> book</a>.
    </p>
  </body>
</html>
```

---

app/views/static_pages/about.html.erb
```html
<!DOCTYPE html>
<html>
  <head>
    <title>About | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>About</h1>
    <p>
      The <a href="http://www.railstutorial.org/"><em>Ruby on Rails
      Tutorial</em></a> is a
      <a href="http://www.railstutorial.org/book">book</a> and
      <a href="http://screencasts.railstutorial.org/">screencast series</a>
      to teach web development with
      <a href="http://rubyonrails.org/">Ruby on Rails</a>.
      This is the sample application for the tutorial.
    </p>
  </body>
</html>
```

---

We got passed
```shell
$ bundle exec rake test
3 tests, 6 assertions, 0 failures, 0 errors, 0 skips
```

---

### 3.4.3 Layouts and embedded Ruby (Refactor)
What are the problems, bad smells?
  - The page titles are almost (but not quite) exactly the same.
  - “Ruby on Rails Tutorial Sample App” is common to all three titles.
  - The entire HTML skeleton structure is repeated on each page.

---
Update the views and extract the common parts

app/views/static_pages/home.html.erb
```html
<% provide(:title, "Home") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Sample App</h1>
    <p>
      This is the home page for the
      <a href="http://www.railstutorial.org/">Ruby on Rails Tutorial</a>
      sample application.
    </p>
  </body>
</html>
```

---

app/views/static_pages/help.html.erb
```html
<% provide(:title, "Help") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>Help</h1>
    <p>
      Get help on the Ruby on Rails Tutorial at the
      <a href="http://www.railstutorial.org/#help">Rails Tutorial help
      section</a>.
      To get help on this sample app, see the
      <a href="http://www.railstutorial.org/book"><em>Ruby on Rails
      Tutorial</em> book</a>.
    </p>
  </body>
</html>
```

---

app/views/static_pages/about.html.erb
```
<% provide(:title, "About") %>
<!DOCTYPE html>
<html>
  <head>
    <title><%= yield(:title) %> | Ruby on Rails Tutorial Sample App</title>
  </head>
  <body>
    <h1>About</h1>
    <p>
      The <a href="http://www.railstutorial.org/"><em>Ruby on Rails
      Tutorial</em></a> is a
      <a href="http://www.railstutorial.org/book">book</a> and
      <a href="http://screencasts.railstutorial.org/">screencast series</a>
      to teach web development with
      <a href="http://rubyonrails.org/">Ruby on Rails</a>.
      This is the sample application for the tutorial.
    </p>
  </body>
</html>
```

---

We should have the <green>GREEN</green>
```shell
$ bundle exec rake test
3 tests, 6 assertions, 0 failures, 0 errors, 0 skips
```

---

### 3.4.4 Setting the root route

## 3.5 Conclusion

### 3.5.1 What we learned in this chapter

## 3.6 Exercises

## 3.7 Advanced testing setup

### 3.7.1 MiniTest reporters

### 3.7.2 Backtrace silencer

### 3.7.3 Automated tests with Guard
