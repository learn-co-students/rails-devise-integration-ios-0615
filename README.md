# Devise

## Introduction

This is a step-by-step tutorial on how to integrate Devise into your Rails app. I integrated this into an example application that I have available [here](https://github.com/irmiller22/blogger/tree/integrate-devise). See the commit logs for each step in the integration process.

## Installing the Gem

Add the `devise` gem to your Gemfile, and bundle install. Once you have done so, you should have access to the gem's command line interface.

## Setting up Devise

In order to add in Devise's authentication tools and resources, run `rails g devise:install` in your Terminal. This will bring up the following message:

```
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. If you are deploying on Heroku with Rails 3.2 only, you may want to set:

       config.assets.initialize_on_precompile = false

     On config/application.rb forcing your application to not access the DB
     or load models when precompiling your assets.

  5. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

===============================================================================
```

These are configurations that you might have to set up if you haven't already. To sum it up: 

- you need to change some configuration settings if you're using a mailer in your application
- you need a root path set up
- you need to include flash messages in `application.html.erb`
- you need to set up asset configurations if you're using Rails 3.2
- you can copy the default Devise views if you're planning on customizing them

## Generating Users

The next step is to generate a User model via Devise. This will set up all of the user-related pages for authorization, such as user registration, log in, reset password, etc.

We'll run `rails g devise User`. This is like `rails g model User`, except that it also sets up the Devise forms as well as the routes. It will generate the following output in your Terminal:

```
      invoke  active_record
      create    db/migrate/20150408135126_devise_create_users.rb
      create    app/models/user.rb
      insert    app/models/user.rb
       route  devise_for :users
```

Let's migrate the database, and then run `rake routes`. You'll see this output:

```
 Prefix Verb   URI Pattern                            Controller#Action
        new_user_session GET    /users/sign_in(.:format)               devise/sessions#new
            user_session POST   /users/sign_in(.:format)               devise/sessions#create
    destroy_user_session DELETE /users/sign_out(.:format)              devise/sessions#destroy
           user_password POST   /users/password(.:format)              devise/passwords#create
       new_user_password GET    /users/password/new(.:format)          devise/passwords#new
      edit_user_password GET    /users/password/edit(.:format)         devise/passwords#edit
                         PATCH  /users/password(.:format)              devise/passwords#update
                         PUT    /users/password(.:format)              devise/passwords#update
cancel_user_registration GET    /users/cancel(.:format)                devise/registrations#cancel
       user_registration POST   /users(.:format)                       devise/registrations#create
   new_user_registration GET    /users/sign_up(.:format)               devise/registrations#new
  edit_user_registration GET    /users/edit(.:format)                  devise/registrations#edit
                         PATCH  /users(.:format)                       devise/registrations#update
                         PUT    /users(.:format)                       devise/registrations#update
                         DELETE /users(.:format)                       devise/registrations#destroy
                         ...
```

Devise, out of the box, gives you access to three controllers: `sessions`, `passwords`, and `registrations`.

It also gives you access to several helper methods: `user_signed_in?`, `current_user`, and `user_session`.

## Requiring Authorization

In order to require your users to either sign up or sign in before using your application, you need a `before_action` filter to redirect your users to a sign in/sign up page. Devise gives you access to one particular filter.

In your `ApplicationController`, put in this line:

```ruby
before_action :authenticate_user!
```

Now when you try to go to any page in your application, it will require you to sign up or sign in.

## Adding Custom Attributes to Sign Up/Sign In Form

What if you wanted to add a `name` attribute to sign up page? There are a couple things that you have to do in order to reflect this addition in your application. In short, you have to re-configure Devise's strong params, generate the standard Devise views in your apps, and make sure that your forms submit properly.

Add this to your `ApplicationController`:

```ruby
before_action :configure_permitted_parameters, if: :devise_controller?

protected

def configure_permitted_parameters
  binding.pry
  devise_parameter_sanitizer.for(:sign_up) << :name
end
```

We have the `binding.pry` in there to check and see what's being permitted through the Devise `RegistrationsController`.

Next step is to generate the views so that we can customize them: `rails g devise:views`. This will output the following:

```
      invoke  Devise::Generators::SharedViewsGenerator
      create    app/views/devise/shared
      create    app/views/devise/shared/_links.html.erb
      invoke  form_for
      create    app/views/devise/confirmations
      create    app/views/devise/confirmations/new.html.erb
      create    app/views/devise/passwords
      create    app/views/devise/passwords/edit.html.erb
      create    app/views/devise/passwords/new.html.erb
      create    app/views/devise/registrations
      create    app/views/devise/registrations/edit.html.erb
      create    app/views/devise/registrations/new.html.erb
      create    app/views/devise/sessions
      create    app/views/devise/sessions/new.html.erb
      create    app/views/devise/unlocks
      create    app/views/devise/unlocks/new.html.erb
      invoke  erb
      create    app/views/devise/mailer
      create    app/views/devise/mailer/confirmation_instructions.html.erb
      create    app/views/devise/mailer/reset_password_instructions.html.erb
      create    app/views/devise/mailer/unlock_instructions.html.erb
```

By default, it'll generate the `confirmations`, `passwords`, `registrations`, `sessions`, and `mailer` views.

The view that we're going to modify is the `registrations/new.html.erb` view, and we're going to add a form field for the `:name` attribute of the User model.

```ruby
<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= devise_error_messages! %>

  <div class="field">
    <%= f.label :name %><br />
    <%= f.text_field :name, autofocus: true %>
  </div>
  ...
<% end %>
```

Then add this to your `application.html.erb`:

```ruby
<body>
  <% if user_signed_in? %>
    <%= current_user.name unless current_user.name.nil? %>
    <%= link_to "Sign Out", destroy_user_session_path, method: :delete %>
  <% else %>
    <%= link_to "Sign In", new_user_session_path %>
  <% end %>

  <%= yield %>

</body>
```

Now you should be able to sign in and sign out effectively, and see the name attribute pop up after you sign up.

## Omniauth with Devise

First thing we'll do is uncomment out the `config.omniauth` configuration line in the `devise.rb` configuration file for Devise.

The next step is to add the `:omniauthable` module and the `:omniauth_providers` to the User model:

```ruby
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable, :omniauthable, :omniauth_providers => [:facebook]
end
```

Then the next step is to get your access token and secret from the Facebook App Development page, available at: https://developers.facebook.com/apps/. I installed the `figaro` gem, and put these into my `application.yml` file:

```yaml
facebook_app_id: ...
facebook_app_secret: ...
```

Then set up the configurations for these keys in your `devise.rb` file:

```ruby
config.omniauth :facebook, ENV["facebook_app_id"], ENV["facebook_app_secret"]
```

Once all of this is set up correctly, you should see this in your routes:

```
user_omniauth_authorize GET|POST /users/auth/:provider(.:format)        devise/omniauth_callbacks#passthru {:provider=>/facebook/}
  user_omniauth_callback GET|POST /users/auth/:action/callback(.:format) devise/omniauth_callbacks#:action
```

Let's add the following to the `application.html.erb` file, and try clicking on it:

```
<%= link_to 'Sign in with Facebook', user_omniauth_authorize_path(:facebook) %>
```

Assuming we have the Oauth configurations set up in our Facebook app (see more [here](http://stackoverflow.com/questions/20910576/how-can-i-add-localhost3000-to-facebook-app-for-development)), it'll redirect to Facebook and ask for permission to use data from your Facebook profile.

The next step is to set up an Omniauth Callbacks controller.

We need to add this to our routes, so that Devise knows how to handle callbacks from Omniauth:

```ruby
devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }
```

Then we'll create the actual controller itself.

```ruby
# app/controllers/users/omniauth_callbacks_controller.rb

class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
end
```

Then let's add a `facebook` method to handle the Omniauth callback from Facebook:

```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    # You need to implement the method below in your model (e.g. app/models/user.rb)
    @user = User.from_omniauth(request.env["omniauth.auth"])

    if @user.persisted?
      sign_in_and_redirect @user, :event => :authentication #this will throw if @user is not activated
      set_flash_message(:notice, :success, :kind => "Facebook") if is_navigational_format?
    else
      session["devise.facebook_data"] = request.env["omniauth.auth"]
      redirect_to new_user_registration_url
    end
  end
end
```

Then the last step is to build out the `from_omniauth` method in the User model.

```ruby
  class User < ActiveRecord::Base
    def self.from_omniauth(auth)
      where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
        user.provider = auth.provider
        user.uid = auth.uid
        user.email = auth.info.email
        user.password = Devise.friendly_token[0,20]
      end
    end
  end
```

Then it should be good to go!

## Resources

- [Devise Documentation](https://github.com/plataformatec/devise)
- [DigitalOcean](https://www.digitalocean.com/) - [Setting Up Devise](https://www.digitalocean.com/community/tutorials/how-to-configure-devise-and-omniauth-for-your-rails-application)
- [Github](http://www.github.com/) - [Omniauth-Facebook](https://github.com/mkdynamic/omniauth-facebook)
