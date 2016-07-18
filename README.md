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
