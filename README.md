# Learning by Doing 
Mackenzie Child's video really inspire me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.           

![image](https://github.com/TimingJL/raddit/blob/master/pic/machenziechild.jpeg)

# How to Build a Reddit or Hacker News Style Web App in Rails     
https://mackenziechild.me/12-in-12/1/       

## What are the main things we want this application to do?
1. User can sign up, sign in , and sign out.
2. Sign in user would able to submit a link with the title and the URL
3. Be able to vote upper down on a application as well as comments on links and submissions.

## Highlights of this course
1. Sign up / Sign in & Out
2. Submit a link with a Title & URL
3. Vote up or down on a link
4. Comment on links submissions

#Create a new rails application
First, I create a rails application named 'raddit'.   

	rails new raddit

Because there is no Javascript interpreter for Rails on Ubuntu Operation System, we have to install `Node.js` or `therubyracer` to get the Javascript interpreter. So we edit the 'Gemfile' under the main folder(raddit folder in this case) and add  
	`gem 'therubyracer'` 
in it. And run bundle install.    

	bundle install

Run rails server to make sure everything was installed correctly.     

	rails server

# Initialize the git and committed, so we can track our work
	git init
	git status
	git add .
	git commit -am ‘Initial Commit’

# Link submissions
## Create a new branch in git
	git checkout -b link_scaffold

## Generate a scaffold
	rails g scaffold Link title:string url:string
	rake db:migrate

## Merge the branch to the master
	git add .
	git commit -am ‘Generate Link Scaffold’
	git checkout master
	git merge link_scaffold
We’d merge the branch into the master branch.

# Create users (Sign in and Sign out)
We use the gem called `devise`. To do this, we start  by creating a new branch

	git checkout -b add_users

To install ‘devise’, we are going to         
https://rubygems.org/gems/devise                                  
We need to add `gem ‘devise’` to our Gemfile

To install, we need to run 

	bundle install

We’re going to run the devise generator

	rails g devise:install


Under `config/environments/development.rb`, we need to add this line

	config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

Under `config/locales/route.rb`, we add 

	root to: "links#index"

So we go back to `http://localhost:3000`, it would show us the links controller as homepage.

In `app/views/layouts/application.html.erb`, we add

	<% flash.each do |name, msg| %>
		<%= content_tag(:div, msg, class: "alert alert-#{name}") %>
	<% end %>

in `<body></body>` before `<%= yield %>`

We add the devise under the views directory

	rails g devise:views

Now we can generate the user with the devise

	rails g devise User

Then generate the model and migration for us

	rake db:migrate

Now, under the model, we have a user. Then, restart the server to check out it actaully worked by going to         

`http://localhost:3000/users/sign_up `       

We can make sure everything is correct by rails console

	rails c

In `rails console`, we do:

	User.count

We can do `SELECT COUNT(*) FROM "users"` by ActiveRecord
If we do

	@User = User.first

We can see user ID, his email, and some other attribute
Then we get out the console by

	exit

or

	control + C

the commit

	git add .
	git commit -am ‘Add devise and create User model’

Now, we have the ability to sign in and out, but it’s not very practical for us to sign in and sign out, so that’s add some links to our view files.      
In our view `app/views/layouts/application.html.erb`, we add conditional statement

	<% if user_signed_in? %>
		<ul>
			<li><%= link_to 'Submit link', new_link_path %></li>
			<li><%= link_to 'Account', edit_user_registration_path %></li>
			<li><%= link_to 'Sign out', destroy_user_session_path, :method => :delete %></li>
		</ul>
	<% else %>
		<ul>
			<li><%= link_to 'Sign up', new_user_registration_path %></li>
			<li><%= link_to 'Sign in', new_user_session_path %></li>
		</ul>		
	<% end %>

and refresh the web page to ensure everything is work.
We hope the sign out account does’t show the summit link. So to fix it, we need to create an association here between a user and links. In our user file `app/models/user.rb`, we are going to add ‘a user has many links’

	has_many :links

And under `app/models/link.rb`, we add ‘a link belongs to user’

	belongs_to :user

So if we jumped into the `rails console`

	@link = Link.first
	@link.user

it will return `nil`.      
If we did’t add `has_many :links` to `user.rb`, it will return `error`.      
So now, the association between the two models is working.      

The issue is that the links table in the database dose’t have a column for users. We gonna need to add a migration, around the migration to add users to links. In the terminal:

	rails g migration add_user_id_to_links user_id:integer:index
	rake db:migrate

That’s go back to rails console to confirm that it worked.

Then, commit

	git add .
	git commit -am ‘Add association between Link and User’

Now, we need to update our link controller so that when a user submits a link that user ID we just created gets assigned to that link.

`app/controllers/links_controller.rb` we gonna change the `new` and the `create` method

	  def new
	    @link = current_user.links.build
	  end

	  def create
	    @link = current_user.links.build(link_params)
		…...
	  end

And now, we need to add some authorization to our controller to make sure nobody can do that not allowed to do. So we should add before filter to our controller.

Under `app/controllers/links_controller.rb`, we add

	before_filter :authenticate_user!, except: [:index, :show]

Now if we are not signed in and hit the destroy, it takes us to a login page which is what we are.
So even though we’ve authenticated, the links still show up. We shouldn’t be able to see them for not signed in.

In our views, `app/views/links/index.html.erb`

        <td><%= link_to 'Show', link %></td>
        <% if link.user == current_user %>
          <td><%= link_to 'Edit', edit_link_path(link) %></td>
          <td><%= link_to 'Destroy', link, method: :delete, data: { confirm: 'Are you sure?' } %></td>
        <% end %>

Finally, we want to get rid of this new link sense, you should be signed in to be able to submit the new link.
`app/views/links/index.html.erb`
we gonna to remove this line:

	<br>
	<%= link_to 'New Link', new_link_path %>

So now, user has to signed in order to create a link as well as add it or destroy that link and only that link the user who created that link is able to destroy that link.

Let’s commit

	git add .
	git commit -am ‘Authorization on links’
	git checkout master
	git merge add_users

So now we have the ability to sign in and out.

# Bootstrap and styling
https://github.com/twbs/bootstrap-sass       

To do that, let’s create a new branch

	git checkout -b add_bootstrap

We need to add bootstrap gem in our `Gemfile`

	gem 'bootstrap-sass', '~> 3.3', '>= 3.3.6'

and that’s do `bundle install`, then go back to our server and restart it.

A few things we to do to get bootstrap working   
Import Bootstrap styles in `app/assets/stylesheets/application.scss`:    
`application.css` needs to rename to `application.scss`

	// "bootstrap-sprockets" must be imported before "bootstrap" and "bootstrap/variables"
	@import "bootstrap-sprockets";
	@import "bootstrap";

Note: If you use `bootstrap-sprockets`, you will get the error:
	```File to import not found or unreadable: bootstrap-sprockets
Only for Twitter Bootstrap 3, bootstrap-sprockets is used.```      
https://rubyplus.com/articles/3981-Integrating-Twitter-Bootstrap-4-with-Rails-5       

And also, Require Bootstrap Javascripts in `app/assets/javascripts/application.js`:

	//= require jquery
	//= require bootstrap-sprockets

It is important that it comes after `//= require jquery`. It is also important that `//= require_tree` is the last thing to be required. The reason is, `//= require_tree` . compiles each of the other Javascript files in the javascripts directory and any subdirectories. If you require `bootstrap-sprockets` after everything else, your other scripts may not have access to the Bootstrap functions.

One things I do know is the scaffold is gonna override currently the bootstrap. So we delete the entire file `app/stylesheets/scaffolds.scss`

## Adding some styling to our pages and our forms
`app/views/layouts/application.html.erb`

	<!DOCTYPE html>
	<html>
	  <head>
	    <title>Raddit</title>
	    <%= csrf_meta_tags %>
	    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
	    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
	  </head>

	  <body>
			<header class="navbar navbar-default" role="navigation">
				<div class="navbar-inner">
				  <div class="container">
				    <div id="logo" class="navbar-brand"><%= link_to "Raddit", root_path %></div>
				      <nav class="collapse navbar-collapse navbar-ex1-collapse">
							<% if user_signed_in? %>
								<ul class="nav navbar-nav navbar-right">
									<li><%= link_to 'Submit link', new_link_path %></li>
									<li><%= link_to 'Account', edit_user_registration_path %></li>
									<li><%= link_to 'Sign out', destroy_user_session_path, :method => :delete %></li>
								</ul>
							<% else %>
								<ul class="nav navbar-nav pull-right">
									<li><%= link_to 'Sign up', new_user_registration_path %></li>
									<li><%= link_to 'Sign in', new_user_session_path %></li>
								</ul>		
							<% end %>	      
				      </nav>	    
				  </div>
				 </div>
			</header>

			<div id="main_content" class="container">
				<% flash.each do |name, msg| %>
					<%= content_tag(:div, msg, class: "alert alert-#{name}") %>
				<% end %>

				<div id="content" class="col-md-9 center-block">
					<%= yield %>
				</div>
			</div>
	  </body>
	</html>

Basically, we just add an navbar with the links.
And we add a bit of sytling to application css file
`app/assets/stylesheets/application.css.scss`

	#logo {
		font-size: 26px;
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: -1px;
		padding: 15px 0;
		a {
			color: #2F363E;
		}
	}

	#main_content {
		#content {
			float: none;
		}
		padding-bottom: 100px;
		.link {
			padding: 2em 1em;
			border-bottom: 1px solid #e9e9e9;
			.title {

				a {
					color: #FF4500;
				}
			}
		}
		.comments_title {
			margin-top: 2em;
		}
		#comments {
			.comment {
				padding: 1em 0;
				border-top: 1px solid #E9E9E9;
				.lead {
					margin-bottom: 0;
				}
			}
		}
	}

It take cares to logo and as well as content and comments and stuff.

Next, in index.html.erb file
`app/views/links/index.html.erb`

	<% @links.each do |link| %>
	  <div class="link row clearfix">
	    <h2>
	      <%= link_to link.title, link %><br>
	      <small class="author">Submitted <%= time_ago_in_words(link.created_at) %> by <%= link.user.email %></small>
	    </h2>
	  </div>
	<% end %>


`app/views/links/show.html.erb`

	<div class="page-header">
	  <h1><a href="<%= @link.url %>"><%= @link.title %></a><br> <small>Submitted by <%= @link.user.email %></small></h1>
	</div>

	<div class="btn-group">
		<%= link_to 'Visit URL', @link.url, class: "btn btn-primary" %>
	</div>

	<% if @link.user == current_user %>
		<div class="btn-group">
			<%= link_to 'Edit',edit_link_path(@link), class: "btn btn-default" %>
			<%= link_to 'Destroy', @link, method: :delete, data: { confirm: 'Are you sure?'}, class: "btn btn-default" %>
		</div>
	<% end %>


`app/views/links/_form.html.erb`

	<%= form_for(link) do |f| %>
	  <% if link.errors.any? %>
	    <div id="error_explanation">
	      <h2><%= pluralize(link.errors.count, "error") %> prohibited this link from being saved:</h2>

	      <ul>
	      <% link.errors.full_messages.each do |message| %>
	        <li><%= message %></li>
	      <% end %>
	      </ul>
	    </div>
	  <% end %>

	  <div class="form-group">
	    <%= f.label :title %><br>
	    <%= f.text_field :title, class: "form-control" %>
	  </div>
	  <div class="form-group">
	    <%= f.label :url %><br>
	    <%= f.text_field :url, class: "form-control" %>
	  </div>
	  <br>
	  <div class="form-group">
	    <%= f.submit "Submit", class: "btn btn-lg btn-primary" %>
	  </div>
	<% end %>


`app/views/devise/registrations/edit.html.erb`

	<h2>Edit <%= resource_name.to_s.humanize %></h2>

	<%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
	  <%= devise_error_messages! %>

	  <div class="panel panel-default">
	    <div class="panel-body">

	      <div class="form-inputs">

	      <div class="form-group">
	        <%= f.label :email %>
	        <%= f.email_field :email, class: "form-control", :autofocus => true %>
	      </div>

	      <div class="form-group">
	        <%= f.label :password %> <i>(leave blank if you don't want to change it)</i>
	        <%= f.password_field :password, class: "form-control", :autocomplete => "off" %>
	      </div>

	      <div class="form-group">
	        <%= f.label :current_password %> <i>(we need your current password to confirm your changes)</i>
	        <%= f.password_field :current_password, class: "form-control" %>
	      </div>

	      </div>

	      <div class="form-group">
	        <%= f.submit "Update", class: "btn btn-primary" %>
	      </div>

	    <% end %>
	  </div>
	  <div class="panel-footer">

	    <h3>Cancel my account</h3>

	    <p>Unhappy? <%= button_to "Cancel my account", registration_path(resource_name), data: { confirm: "Are you sure?" }, method: :delete, class: "btn btn-default" %></p>

	  </div>

`app/views/devise/registrations/new.html.erb`

	<h2>Sign up</h2>

	<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
	  <%= devise_error_messages! %>

	    <div class="form-group">
	      <%= f.label :email %>
	      <%= f.email_field :email, autofocus: true, class: "form-control", required: true %>
	    </div>

	    <div class="form-group">
	      <%= f.label :password %>
	      <%= f.password_field :password, class: "form-control", required: true %>
	    </div>

	    <div class="form-group">
	      <%= f.label :password_confirmation %>
	      <%= f.password_field :password_confirmation, class: "form-control", required: true %>
	    </div>

	    <div class="form-group">
	      <%= f.submit "Sign up", class: "btn btn-lg btn-primary" %>
	    </div>
	<% end %>

	<%= render "devise/shared/links" %>


`app/views/devise/sessions/new.html.erb`

	<h2>Sign in</h2>

	<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>

	  <div class="form-group">
	    <%= f.label :email %>
	    <%= f.email_field :email, autofocus: true, class: "form-control", required: false %>
	  </div>

	  <div class="form-group">
	    <%= f.label :password %>
	    <%= f.password_field :password, class: "form-control", required: false %>
	  </div>

	  <div class="checkbox">
	    <label>
	    <%= f.check_box :remember_me, required: false, as: :boolean if devise_mapping.rememberable? %> Remember Me
	    </label>
	  </div>

	  <div class="form-group">
	    <%= f.submit "Sign In", class: "btn btn-primary" %>
	  </div>

	<% end %>

	<%= render "devise/shared/links" %>


Now, everything looks very good. Let’s commit and emerge

	git add .
	git commit -am ‘Add structure and basic styling’
	git checkout master
	git merge add_bootstrap


# Voting

	git checkout -b add_acts_as_votable
We need to use acts_as_votable gem to build the voting
so we add `gem 'acts_as_votable', '~> 0.10.0'` in the `Gemfile` and 

	bundle install

then restart the server
So now we need to generate and run the migration on the acts_as_votable gem

	rails g acts_as_votable:migration

We’d create the database migration

	rake db:migrate

The first thing we need to do is in our model, we need to add `acts_as_votable` before the `belongs_to :user` in `app/models/link.rb`

	acts_as_votable

Let’s going to the rails console to confirm that is working

	rails c
	@link = Link.first
	@user = User.first
	@link.liked_by @user
	@link.votes_for.size
	@link.save

So to get this inside our views, we need to first had some routes for links

`config/routes.rb`

	Rails.application.routes.draw do
	  devise_for :users
	  resources :links do 
	  	member do 
	      put "like", to:    "links#upvote"
	      put "dislike", to: "links#downvote"
	  	end
	  end  

	  root to: "links#index"
	end

in `app/controller/links_controller.rb`, we need to create the up vote and down vote method

	  def upvote
	    @link = Link.find(params[:id])
	    @link.upvote_by current_user
	    redirect_to :back
	  end

	  def downvote
	    @link = Link.find(params[:id])
	    @link.downvote_by current_user
	    redirect_to :back
	  end

and inside the view `app/views/links/index.html.erb`

	<% @links.each do |link| %>
	  <div class="link row clearfix">
	    <h2>
	      <%= link_to link.title, link %><br>
	      <small class="author">Submitted <%= time_ago_in_words(link.created_at) %> by <%= link.user.email %></small>
	    </h2>

	    <div class="btn-group">
	        <a class="btn btn-default btn-sm" href="<%= link.url %>">Visit Link</a>
	        <%= link_to like_link_path(link), method: :put, class: "btn btn-default btn-sm" do %>
	          <span class="glyphicon glyphicon-chevron-up"></span>
	          Upvote
	          <%= link.get_upvotes.size %>
	        <% end %>
	        <%= link_to dislike_link_path(link), method: :put, class: "btn btn-default btn-sm" do %>
	          <span class="glyphicon glyphicon-chevron-down">
	          Downvote
	          <%= link.get_downvotes.size %>
	        <% end %>
	      </div>
	    </div>
	<% end %>

So we can Upvote and Downvote correctly     

Then, we add Upvote and Downvote in the show page
In `app/views/links/show.html.erb`  , we add the following code at the bottom

	<div class="btn-group pull-right">
	  <%= link_to like_link_path(@link), method: :put, class: "btn btn-default btn-sm" do %>
	    <span class="glyphicon glyphicon-chevron-up"></span>
	    Upvote
	    <%= @link.get_upvotes.size %>
	  <% end %>
	  <%= link_to dislike_link_path(@link), method: :put, class: "btn btn-default btn-sm" do %>
	    <span class="glyphicon glyphicon-chevron-down">
	    Downvote
	    <%= @link.get_downvotes.size %>
	  <% end %>
	</div>

Let’s create a new user `Sign up`, then we can Upvote and Downvote again. 
And the `edit` and `destroy` links are gone because this is not my post.
Then commit

	git add .
	git commit -am ‘Added and setup acts_as_votable’
	git checkout master
	git merge add_acts_as_votable

# Comment on boats
We’re gonna make it so we can comment and the person who made that comment is able to delete the comment. So let’s create a new branch to do this.

	git checkout -b add_comments

To implemente it through the gem, we are going to create a scaffold	and do it ourself.

	rails g scaffold Comment link_id:integer:index body:text user:references --skip-stylesheets

Let’s create migration model as well as the views and stuff for us which is fantastic.

	rake db:migrate

## Simple_form
We’re gonna to user a gem called `simple_form` to make it easier. We paste the following gem to the `Gemfile` and run bundle install

	gem 'simple_form', '~> 3.2', '>= 3.2.1'
	bundle update
	bundle install

Then, first thing we need to do is add association between our comments and our links model.
`app/models/link.rb`

	class Link < ApplicationRecord
		acts_as_votable
		belongs_to :user
		has_many :comments
	end

`app/models/comment.rb`

	class Comment < ApplicationRecord
	  belongs_to :user
	  belongs_to :link
	end

Next, we need to add resource to our routes
`config/routes.rb`

	Rails.application.routes.draw do
	  resources :comments
	  devise_for :users
	  resources :links do 
	  	member do 
	      put "like", to:    "links#upvote"
	      put "dislike", to: "links#downvote"
	  	end
	  	resources :comments
	  end  

	  root to: "links#index"
	end

You can see the routes

	rake routes
We attempt to add the comment routes for us. We need to update comments controller.
In `app/controllers/commnets_controller.rb`, we need to update `create method`

	  def create
	    @link = Link.find(params[:link_id])
	    @comment = @link.comments.new(comment_params)
	    @comment.user = current_user

	    respond_to do |format|
	      if @comment.save
	        format.html { redirect_to @link, notice: 'Comment was successfully created.' }
	        format.json { render json: @comment, status: :created, location: @comment }
	      else
	        format.html { render action: "new" }
	        format.json { render json: @comment.errors, status: :unprocessable_entity }
	      end
	    end
	  end


Now we need to add a form to our show page. We add the code at the bottom.
`app/views/links/show.html.erb`

	<h3 class="comments_title">
	  <%= @link.comments.count %> Comments
	</h3>

	<div id="comments">
	  <%= render :partial => @link.comments %>
	</div>
	<%= simple_form_for [@link, Comment.new]  do |f| %>
	  <div class="field">
	    <%= f.text_area :body, class: "form-control" %>
	  </div>
	  <br>
	  <%= f.submit "Add Comment", class: "btn btn-primary" %>
	<% end %>

on the comment folder, new a file:
`app/views/comments/_comment.html.erb`

	<%= div_for(comment) do %>
		<div class="comments_wrapper clearfix">
			<div class="pull-left">
				<p class="lead"><%= comment.body %></p>
				<p><small>Submitted <strong><%= time_ago_in_words(comment.created_at) %> ago</strong> by <%= comment.user.email %></small></p>
			</div>

			<div class="btn-group pull-right">
				<% if comment.user == current_user -%>
					<%= link_to 'Destroy', comment, method: :delete, data: { confirm: 'Are you sure?' }, class: "btn btn-sm btn-default" %>
				<% end %>
			</div>
		</div>
	<% end %>

We go back to the application reload it, and go to the show page, you can see the comment form is showing up.
## If you getting NoMethodError in Links#show 
The `div_for` method has been removed from Rails. To continue using it, add the `record_tag_helper` gem to your Gemfile:

  gem 'record_tag_helper', '~> 1.0'
Consult the Rails upgrade guide for details.

	gem 'record_tag_helper', '~> 1.0'
	bundle update
	bundle install
	restart the server

Let’s go ahead and commit and merge what we just did.

	git add .
	git commit -am ‘Add comments’
	git checkout master
	git merge add_comments

So the website is pretty much done. But one thing that’s been bugging me I wanna take care of it’s the I’m submitted by email instead of by name. So I wanna add name field to the users. We create a new branch for this.

	git checkout -b ‘add_name_to_users’

To do this, we need to do migration

	rails g migration add_name_to_users name:string

So that creates our migration file

	rake db:migrate

Now we need to add input to our form field in order to accept that.
`app/views/devise/registrations/edit.html.erb`

      <div class="form-group">
        <%= f.label :name %>
        <%= f.text_field :name, class: "form-control", :autofocus => true %>
      </div>   

Then we need to add some code to our application controller to save the name data.
`app/controllers/application_controller.rb`

	class ApplicationController < ActionController::Base
	  # Prevent CSRF attacks by raising an exception.
	  # For APIs, you may want to use :null_session instead.
	  protect_from_forgery with: :exception
		before_filter :configure_permitted_parameters, if: :devise_controller?

		protected

	  def configure_permitted_parameters
	    devise_parameter_sanitizer.for(:sign_up) << :name
	    devise_parameter_sanitizer.for(:account_update) << :name
	  end

	end

## Getting undefined method `simple_form_for' in RoR app
http://stackoverflow.com/questions/19791531/how-to-specify-devise-parameter-sanitizer-for-edit-action      
The ` .for` method is deprecated, now we use ` .permit`
The first arg is the action name. ` :sign_up` is for creating new Devise resources (such as users), and` :account_update` is for editing/updating the resource.   
The second arg, ` :keys` contains an array of the parameters you allow.    
If you want `nested_attributes`, there is an example in ` :account_update`, you put a separate array in with the key being ` <object>_attributes`.

	class ApplicationController < ActionController::Base
	  # Prevent CSRF attacks by raising an exception.
	  # For APIs, you may want to use :null_session instead.
	  protect_from_forgery with: :exception
	  before_filter :configure_permitted_parameters, if: :devise_controller?

		protected

	  def configure_permitted_parameters
	    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
	    devise_parameter_sanitizer.permit(:account_update, keys: [:name])
	  end

	end


`app/views/links/index.html.erb`
change

    <h2>
      <%= link_to link.title, link %><br>
      <small class="author">Submitted <%= time_ago_in_words(link.created_at) %> by <%= link.user.email %></small>
    </h2>

to

    <h2>
      <%= link_to link.title, link %><br>
      <small class="author">Submitted <%= time_ago_in_words(link.created_at) %> by <%= link.user.name %></small>
    </h2>

Now the same thing to the show page
`app/views/links/show.html.erb`

change the `link.user.email` to `link.user.name`

And as well as comment
`app/views/comments/_comment.html.erb`
change the `link.user.email` to `link.user.name`

A new user sign up, the form automatically gets the name
In `app/views/devise/registrations/new.html.erb` add:

    <div class="form-group">
      <%= f.label :name %>
      <%= f.text_field :name, class: "form-control", :autofocus => true %>
    </div> 

Finished!
