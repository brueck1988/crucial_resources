# Rails Database Development Checklist 

### Warning: This checklist does not rely on TDD to build up the rails app.  It uses some MVC code snippets, so make sure to replace them with names you use on your project.  Most references are from               
    https://backend.turing.io/module2/misc/blogger
    https://github.com/turingschool-examples/task_manager_rails

1. Create Rails Application 
 * From the command line, start a new rails app.  For example,  
   ```ruby
   rails new project_name -T -d="postgresql" --skip-spring --skip-turbolinks

   ```
   (For a new api use --api flag)
2. Environment Setup 
 *Install in Gemfile near top:
    ```ruby
    gem 'jbuilder', '~> 2.5'
    gem 'fast_jsonapi'
    gem 'faraday'
    gem 'figaro'
    gem 'bcrypt', '~> 3.1.7'
    ```
 * Install all necessary gems in Gemfile under `group :development, :test do`.  For example, 
   ```ruby 
   gem 'pry'
   gem 'rspec-rails', '~> 4.0.1'
   gem 'capybara'
   gem 'launchy'
   gem 'simplecov'
   gem 'shoulda-matchers', '~> 3.1'
   gem 'orderly'
   ```
   * Install all necessary gems in Gemfile under `group :test do`.  For example, 
     ```ruby 
     gem 'webmock'
     gem 'vcr'
     ```
     (Also need to add "require 'webmock/rspec'" to spec/spec_helper.)
     For VCR, we need to add to spec/rails_helper:
     ```ruby
    VCR.configure do |config|
      config.cassette_library_dir = 'spec/fixtures/vcr_cassettes'
      config.hook_into :webmock
      #config.filter_sensitive_data('DONT_SHARE_MY_PROPUBLIC_SECRET_KEY') { ENV['PROPUBLICA_KEY'] }   #Update per API
      config.default_cassette_options = { re_record_interval: 7.days }
      config.configure_rspec_metadata!
    end
    ```
 * Run the following command in the terminal to install RSpec run: 
   ```ruby
   rails g rspec:install
   ```
 * Add the following simplecov configuration code at the top of the `/spec/rails-helper.rb` file
    ```ruby
    require 'simplecov'
    SimpleCov.start
    ```
 * Add the following shoulda-matchers configuration code into the `/spec/rails-helper.rb` file
    ```ruby
    Shoulda::Matchers.configure do |config|
      config.integrate do |with|
         with.test_framework :rspec
         with.library :rails
      end
    end
    ```
    
 * From the command line run: 
   ```
   bundle update
   ```
 * From the command line run: 
   ```
   bundle install
   ``` 
3. Create Database and Resources 
 * To create the database in the terminal run: 
   ```
   rails db:create
   ```
 * If you get a warning that Database already exist, make sure you DROP the table then create. in the terminal run: 
   ```
   rails db:drop
   rails db:create
   ```

 * To generate resources in the terminal run 
   * For no relationship (aka no foreign keys), for example,    
   ```
   rails generate migration CreateArticles title:string body:text
   ```
   * For a belongs to relationship, for example, 
   ``` 
   rails generate migration CreateComments author_name:string body:text article:references
   ```
   * If you need to add a relationship after you migrated, 
   ```
   rails generate migration AddCommentsToArticles article:references
   ```
 * Check the migration that was generated to make sure the table is being created properly and to add timestamps 
   ```ruby 
   t.timestamps
   ```
 * After a migration is successfully generated, run `rails db:migrate` in the terminal. 

###Models
 * One-to-Many  
   * Create a model by creating a model file in `app/models/`, for example `app/models/comment.rb`.  For example the        belongs to side of the relationship, 
     ```ruby 
     class Comment < ApplicationRecord  
        belongs_to :article   
     end
     ```
   * Create a model by creating a model file in `app/models/`, for example `app/models/article.rb`.  For example the        many side of the relationship,
     ```ruby 
     class Article < ActiveRecord::Base
        has_many :comments
     end
     ```
   * Create a test by creating a file in `spec/models/`, for the belongs to side of the relationship, for example            `spec/models/comment_spec.rb`. For example, 
     ```ruby 
     require "rails_helper"

     describe Comment, type: :model do
        describe "relationships" do
           it {should belong_to(:article)}
        end
     end
     ```
    * Create a test by creating a file in `spec/models/`, for the many side of the relationship, for example      
      `spec/models/article_spec.rb`.  For example, 
      ```ruby 
      require "rails_helper"

      describe Article, type: :model do
         describe "validations" do
            it {should have_many(:comments)}
         end
      end
      ```
 * Many-to-Many
   * When referring to a camel cased table with multiple words, for example `SongArtists`, in a test or model, use          snake case, for example `song_artists`.  
   * Create a model in `app/models/`, for example, `app/models/tag.rb`, or add code if it has already been created          for each side of the many-to-many relationship.  Make sure to do this for each of the many-to-many models. For          example the many side of the relationship,
      ```ruby 
      class Tag < ApplicationRecord
         has_many :taggings
         has_many :articles, through: :taggings
      end  
      ```
   * Create a join model in `app/models/`, for example, `app/models/tagging.rb`.  For example the join model, 
      ```ruby 
      class Tagging < ApplicationRecord
         belongs_to :tag
         belongs_to :article
      end 
      ```
   * Create a test for each many side of the relationship, for example, `spec/models/tag_spec.rb`.  For example,     
      ```ruby 
      require "rails_helper"

      describe Tag, type: :model do
         describe "relationships" do
            it {should have_many(:tagings)}
            it {should have_many(:articles).through(:taggings)}
         end
      end
      ```
   * Create a test for the joins, for example, `spec/models/tagging_spec.rb`.  For example, 
      ```ruby
      require "rails_helper"
      
      describe Tagging, type: :model do
         describe "relationships" do
            it {should belong_to(:tag)}
            it {should belong_to(:article)}
         end
      end
      ```
 
 * Relationships 
   1. One-to-Many 
    * The objects on the "many" end should have a foreign key referencing the "one" object.
   2. Many-to-Many
    * * The joins table should have a foreign key referencing each of the many tables. 


### Edit table column data type
When you need to make an edit, AFTER you have migrated, you SHOULD create a NEW migration.
1. Run generate migration in the terminal.  For example, 
   ```ruby 
   rails generate migration ChangeBodyToBeTextInComments
   ```
2. Open migration file and put in change, for example,
   ```ruby 
   change_column :comments, :body, :text
   ```
3. `rails db:migrate`
4. Check shcema file that the column data type has changed.  

### Add column to table 
1. Run generate migration in the terminal.  For example, 
   ```ruby 
   rails generate migration AddEmailToComments
   ```
2. Open migration file and put in change, for example,
   ```ruby 
   add_column :comments, :email, :string
   ```
3. `rails db:migrate`
4. Check shcema file that the column data type has changed.  


 
 ## Relationships 
    1. One-to-Many 
    * The objects on the "many" end should have a foreign key referencing the "one" object.
    * Objects on the "many" end "belong_to" the object on the "one" end. should be singualar.  For example, "an article has_many comments and a comment belongs_to an article"  
    *  Belongs to should be singualr
    *  has many should be plural

   2. Many-to-Many
    * The joins table should have a foreign key referencing each of the many tables.     
4. Testing 
 * Create both a `models` and `features` sub-directory in the `spec` folder.
 * Framework for a model test,
   ```ruby 
   require "rails_helper"

   describe Article, type: :model do
      describe "validations" do
         it {should validate_presence_of(:title)}
         it {should validate_presence_of(:body)}
      end
   end
   ```
 * Framework for a feature test, 
   ```ruby 
   require "rails_helper"

   RSpec.describe "articles index page", type: :feature do
      before :each do
         @article_1 = Article.create!(title: "Title 1", body: "Body 1")
         @article_2 = Article.create!(title: "Title 2", body: "Body 2")
      end
   end

   describe "user sees all articles" do
      describe "they visit /articles" do
         it "displays all articles" do

            visit '/articles'

            expect(page).to have_content(article_1.title)
            expect(page).to have_content(article_2.title)
         end 
      end
   end
   ```
   5. Create Routes
 * Add code into `/config/routes.rb`, such as one of the following examples,
   ```ruby 
   resources :articles 
   resources :articles, only: [:index, :show]
   get "/articles/:id", to: "articles#show"
   ```
 * Uses snake_case for controller names in the routes if it more than one word.  Example, `monster_trucks#show` for a      controller named `MonsterTrucksController`.
 * Seven RESTful routes: `index, new, create, show, edit, update, and destory` 
    - `get '/articles', to: "articles#index` => *display a list of all resources*
    - `get '/articles/new', to: "articles#new`=> *show form to make a new resource*
    - `post '/articles', to: "articles#create"` => *add new resource to the database, then redirect*
    - `get '/articles/:id', to: "articles#show"` => *show information about a particular resource* 
    - `get '/articles/:id/edit', to: "articles#edit"`=> *show from to edit an existing resource* 
    - `patch '/articles/:id', to: "articles#update"`=> *update an existing resource, then redirect* 
    - `delete '/articles/:id', to: "articles#destroy"` => *delete a particular resource, then redirect*
 * To display all routes from the command line run 
   ```
   rails routes
   ```
6. Create Controllers 
 * Create a controller file in `/app/controllers/`, for example `app/controllers/articles_controller.rb` 
 * Add in the framework for a controller, for example
   ```ruby 
   class ArticlesController < ApplicationController

   end
   ```
7. Create Views
 * Create directory under `app/views` named after the controller, for example, `app/views/articles`, and with a file      for the view, for example, `index.html.erb`. should look like `app/views/articles/index.html.erb`
---

### form_with
* For a search 
    ```ruby 
    <%= form_with(url: "/search", method: "get") do %>
      <%= label_tag(:q, "Search for:") %>
      <%= text_field_tag(:q) %>
      <%= submit_tag("Search") %>
    <% end %>
    ```
* For a create, using a model url
    ```ruby
    <%= form_with url (path), local: true do |f| %>
      <%= f.label :name %>
      <%= f.text_field :name %>

      <%= f.label :breed %>
      <%= f.text_field :breed %>

      <%= f.label :age %>
      <%= f.number_field :age %>

      <%= f.submit %>
    <% end %>
    ```
### validates_presence_of 
* Use validates_presence_of in model.  For example, 
  ```ruby 
  validates_presence_of :title, :body, presence: true 
  ```
* Use validates_presence_of in model tests.  For example, 
  ```ruby 
  describe "validations" do
    it {should validate_presence_of :title}
  end 
  ```
### link_to
* Use path helper with verb if the path helper is ambiguous.  For example, 
  ```ruby 
  <%= link_to "Delete", article_path(@article), method: :delete %>
  ```