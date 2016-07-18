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
