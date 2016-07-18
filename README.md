# Learning by Doing 
Mackenzie Child's video really inspire me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.


# How to Build a Reddit or Hacker News Style Web App in Rails

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

Because there is no Javascript interpreter for Rails on Ubuntu Operation System, we have to install `Node.js` or `therubyracer` to get the Javascript interpreter. So we edit the 'Gemfile' under the main folder(raddit folder in this case) and add  `gem 'therubyracer'` in it. And run bundle install.
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
we’d merge the branch into the master branch

# Create users (Sign in and Sign out)
we use the gem called `devise`. To do this, we start  by creating a new branch
	git checkout -b add_users
To install ‘devise’, we are going to 
https://rubygems.org/gems/devise                                  
we need to add `gem ‘devise’` to our Gemfile

To install, we need to run 
	bundle install

we’re going to run the devise generator
	rails g devise:install


under `config/environments/development.rb`, we need to add this line
	config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

under `config/locales/route.rb`, we add `root to: "links#index"`
so we go back to `http://localhost:3000` it would show us the links controller as homepage.

In `app/views/layouts/application.html.erb`, we add
	<% flash.each do |name, msg| %>
		<%= content_tag(:div, msg, class: "alert alert-#{name}") %>
	<% end %>
in <body></body> before `<%= yield%>`

we add the devise under the views directory
rails g devise:views
now we can generate the user with the devise
	rails g devise User
then generate the model and migration for us
	rake db:migrate

Now, under the model, we have a user. Then, restart the server to check out it actaully worked by going to 
http://localhost:3000/users/sign_up         

we can make sure everything is correct by rails console
	rails c
In rails console, we do:
	User.count
we can do `SELECT COUNT(*) FROM "users"` by ActiveRecord
If we do
	@User = User.first
we can see user ID, his email, and some other attribute
Then we get out the console by
	exit
or
	control + C

the commit
	git add .
	git commit -am ‘Add devise and create User model’

Now, we have the ability to sign in and out, but it’s not very practical for us to sign in and sign out, so that’s add some links to our view files.
In our view `app/views/layouts/application.html.erb`. we add conditional statement
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

and refresh the web page to ensure everythings work.
We hope the sign out account does’t show the summit lin. So to fix it, we need to create an association here between a user and links. In our user file `app/models/user.rb`, we are going to add ‘a user has many links’
	has_many :links
And under ‘app/models/link.rb’, we add ‘a link belongs to user’
	belongs_to :user
So if we jumped into the rails console
	@link = Link.first
	@link.user
it will return `nil`. If we did’t add `has_many :links` to `user.rb`, it will return error.
So now, the association between the two models is working.
The issue is that the links table in teh database dose’t have a column for users. We gonna need to add a migration, around the migration to add users to links. In the terminal:
	rails g migration add_user_id_to_links user_id:integer:index
	rake db:migrate
That’s go back to rails console to confirm that it worked.

Then, commit
	git add .
	git commit -am ‘Add association between Link and User’

Now, we need to update our link controller so that when a user submits a link that user ID we just created gets assigned to that link.

`app/controllers/links_controller.rb` we gonna change the new and the create method
	  def new
	    @link = current_user.links.build
	  end

	  def create
	    @link = current_user.links.build(link_params)
		…...
	  end

And now, we need to add some authorization to our controller to make sure nobody can do that not allowed to do. So we should add before filter to our controller
under `app/controllers/links_controller.rb`, we add
before_filter :authenticate_user!, except: [:index, :show]
now if we are not signed in and hit the destroy, it takes us to a login page which is what we are.
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
so now, user has to signed in order to create a link as well as add it or destroy that link and only that link the user who created that link is able to destroy that link.
That’s commit
	git add .
	git commit -am ‘Authorization on links’
	git checkout master
	git merge add_users
So now we have the ability to sign in and out.

# Bootstrap and styling
https://github.com/twbs/bootstrap-sass           
To do that, let’s create a new branch
	git checkout -b add_bootstrap
we need to add bootstrap gem in our `Gemfile`
 gem 'bootstrap-sass', '~> 3.3', '>= 3.3.6'
and that’s do `bundle install`, then go back to our server and restart it.
A few things we to do to get bootstrap working
Import Bootstrap styles in `app/assets/stylesheets/application.scss`:
`application.css` needs to rename to `application.scss`
	// "bootstrap-sprockets" must be imported before "bootstrap" and "bootstrap/variables"
	@import "bootstrap-sprockets";
	@import "bootstrap";
Note: If you use `bootstrap-sprockets`, you will get the error:
	File to import not found or unreadable: bootstrap-sprockets
Only for Twitter Bootstrap 3, bootstrap-sprockets is used.
https://rubyplus.com/articles/3981-Integrating-Twitter-Bootstrap-4-with-Rails-5       

And also, Require Bootstrap Javascripts in `app/assets/javascripts/application.js`:
	//= require jquery
	//= require bootstrap-sprockets

It is important that it comes after //= require jquery. It is also important that //= require_tree is the last thing to be required. The reason is, //= require_tree . compiles each of the other Javascript files in the javascripts directory and any subdirectories. If you require bootstrap-sprockets after everything else, your other scripts may not have access to the Bootstrap functions.

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
	  <h1><a href="<%= @link.url %>"><%= @link.title %></a><br> <small>Submitted by <%= @link.user_id %></small></h1>
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
