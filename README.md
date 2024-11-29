# README

This README would normally document whatever steps are necessary to get the
application up and running.

## Folder structure
|File/Folder|Purpose|
|-----------|-------|
|app/|Contains the controllers, models, views, helpers, mailers, jobs, and assets for your application.|
|bin/|Contains the rails script that starts your app and can contain other scripts you use to set up, update, deploy, or run your application.|
|config/|Contains configuration for your application's routes, database, and more. This is covered in more detail in [Configuring Rails Applications](https://guides.rubyonrails.org/configuring.html).|
|config.ru|Rack configuration for Rack-based servers used to start the application. For more information about Rack, see the [Rack website](https://rack.github.io/).|
|db/|Contains your current database schema, as well as the database migrations.|
|Dockerfile|Configuration file for Docker.|
|Gemfile/ Gemfile.lock| These files allow you to specify what gem dependencies are needed for your Rails application. These files are used by the Bundler gem.|
|lib/|Extended modules for your application.|
|log/|Application log files.|
|public/|Contains static files and compiled assets. When your app is running, this directory will be exposed as-is.|
|Rakefile|This file locates and loads tasks that can be run from the command line. The task definitions are defined throughout the components of Rails. Rather than changing Rakefile, you should add your own tasks by adding files to the lib/tasks directory of your application.|
|script/|Contains one-off or general purpose [scripts](https://github.com/rails/rails/blob/main/railties/lib/rails/generators/rails/script/USAGE) and [benchmarks](https://github.com/rails/rails/blob/main/railties/lib/rails/generators/rails/benchmark/USAGE).|
|storage/|Active Storage files for Disk Service. This is covered in [Active Storage Overview](https://guides.rubyonrails.org/active_storage_overview.html).|
|test/|Unit tests, fixtures, and other test apparatus. These are covered in [Testing Rails Applications](https://guides.rubyonrails.org/testing.html).|
|tmp/|Temporary files (like cache and pid files).|
|vendor/|A place for all third-party code. In a typical Rails application this includes vendored gems.|
|


## AutoLoading
Rails applications do not use require to load application code.

You may have noticed that `ArticlesController` inherits from `ApplicationController`, but `app/controllers/articles_controller.rb` does not have anything like

```ruby
require "application_controller" # DON'T DO THIS.
```

Application classes and modules are available everywhere, you do not need and ***should not*** load anything under `app` with `require`. This feature is called autoloading, and you can learn more about it in [Autoloading and Reloading Constants](https://guides.rubyonrails.org/autoloading_and_reloading_constants.html).

You only need `require` calls for two use cases:

- To load files under the `lib` directory.
- To load gem dependencies that have `require: false` in the `Gemfile`.

## MVC and You
So far, we've discussed routes, controllers, actions, and views. All of these are typical pieces of a web application that follows the [MVC (Model-View-Controller)](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) pattern. MVC is a design pattern that divides the responsibilities of an application to make it easier to reason about. Rails follows this design pattern by convention.

Since we have a controller and a view to work with, let's generate the next piece: a model.

### Generating a model
A *model* is a Ruby class that is used to represent data. Additionally, models can interact with the application's database through a feature of Rails called Active Record.

To define a model, we will use the model generator:

```ruby
bin/rails generate model Article title:string body:text
```

### Database migrations
*Migrations* are used to alter the structure of an application's database. In Rails applications, migrations are written in Ruby so that they can be database-agnostic.

Let's take a look at the contents of our new migration file:

```ruby
class CreateArticles < ActiveRecord::Migration[8.0]
  def change
    create_table :articles do |t|
      t.string :title
      t.text :body

      t.timestamps
    end
  end
end
```

The call to `create_table` specifies how the articles table should be constructed. By default, the `create_table` method adds an `id` column as an auto-incrementing primary key. So the first record in the table will have an `id` of 1, the next record will have an `id` of 2, and so on.

Inside the block for `create_table`, two columns are defined: `title` and `body`. These were added by the generator because we included them in our generate command (`bin/rails generate model Article title:string body:text`).

On the last line of the block is a call to `t.timestamps`. This method defines two additional columns named `created_at` and `updated_at`. As we will see, Rails will manage these for us, setting the values when we create or update a model object.

Let's run our migration with the following command:

```ruby
bin/rails db:migrate
```

### Using a model to interact with the database
To play with our model a bit, we're going to use a feature of Rails called the console. The console is an interactive coding environment just like irb, but it also automatically loads Rails and our application code.

Let's launch the console with this command:

```ruby
bin/rails console
```

You should see a rails console prompt like:

```ruby
Loading development environment (Rails 8.0.0)
blog(dev)>
```

At this prompt, we can initialize a new Article object:

```ruby
blog(dev)> article = Article.new(title: "Hello Rails", body: "I am on Rails!")
```

It's important to note that we have only initialized this object. This object is not saved to the database at all. It's only available in the console at the moment. To save the object to the database, we must call `save`:

```ruby
blog(dev)> article.save
(0.1ms)  begin transaction
Article Create (0.4ms)  INSERT INTO "articles" ("title", "body", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["title", "Hello Rails"], ["body", "I am on Rails!"], ["created_at", "2020-01-18 23:47:30.734416"], ["updated_at", "2020-01-18 23:47:30.734416"]]
(0.9ms)  commit transaction
=> true
```

when we want to fetch all articles from the database, we can call all on the model:

```ruby
blog(dev)> Article.all
=> #<ActiveRecord::Relation [#<Article id: 1, title: "Hello Rails", body: "I am on Rails!", created_at: "2020-01-18 23:47:30", updated_at: "2020-01-18 23:47:30">]>
```
This method returns an ActiveRecord::Relation object, which you can think of as a super-powered array.

> **Tip:** To learn more about models, see [Active Record Basics](https://guides.rubyonrails.org/active_record_basics.html) and [Active Record Query Interface](https://guides.rubyonrails.org/active_record_querying.html).

### Showing a list of articles
Let's go back to our controller in `app/controllers/articles_controller.rb`, and change the index action to fetch all articles from the database:

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end
end
```

Controller instance variables can be accessed by the view. That means we can reference `@articles` in `app/views/articles/index.html.erb`. Let's open that file, and replace its contents with:

```erb
<h1>Articles</h1>

<ul>
  <% @articles.each do |article| %>
    <li>
      <%= article.title %>
    </li>
  <% end %>
</ul>
```

The above code is a mixture of HTML and ERB. ERB, short for [Embedded Ruby](https://docs.ruby-lang.org/en/3.2/ERB.html), is a templating system that evaluates Ruby code embedded in a document. Here, we can see two types of ERB tags: `<% %>` and `<%= %>`. The `<% %>` tag means "evaluate the enclosed Ruby code." The `<%= %>` tag means "evaluate the enclosed Ruby code, and output the value it returns." Anything you could write in a regular Ruby program can go inside these ERB tags, though it's usually best to keep the contents of ERB tags short, for readability.

Since we don't want to output the value returned by `@articles.each`, we've enclosed that code in `<% %>`. But, since we do want to output the value returned by `article.title` (for each article), we've enclosed that code in `<%= %>`.

## CRUDit Where CRUDit Is Due
Almost all web applications involve CRUD (Create, Read, Update, and Delete) operations. You may even find that the majority of the work your application does is CRUD. Rails acknowledges this, and provides many features to help simplify code doing CRUD.

### Showing a single article
We currently have a view that lists all articles in our database. Let's add a new view that shows the title and body of a single article.

We start by adding a new route that maps to a new controller action (which we will add next). Open config/routes.rb, and insert the last route shown here:

```ruby
Rails.application.routes.draw do
  root "articles#index"

  get "/articles", to: "articles#index"
  get "/articles/:id", to: "articles#show"
end
```

The new route is another `get` route, but it has something extra in its path: `:id`. This designates a route parameter. A route parameter captures a segment of the request's path, and puts that value into the params Hash, which is accessible by the controller action. For example, when handling a request like GET `http://localhost:3000/articles/1`, 1 would be captured as the value for `:id`, which would then be accessible as `params[:id]` in the show action of `ArticlesController`.

Let's add that `show` action now, below the index action in `app/controllers/articles_controller.rb`:

```ruby
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end
end
```

The show action calls `Article.find` with the `ID` captured by the route parameter. The returned article is stored in the `@article` instance variable, so it is accessible by the view. By default, the show action will render `app/views/articles/show.html.erb`.



## Things you may want to cover:

* Ruby version

* System dependencies

* Configuration

* Database creation

* Database initialization

* How to run the test suite

* Services (job queues, cache servers, search engines, etc.)

* Deployment instructions

* ...
