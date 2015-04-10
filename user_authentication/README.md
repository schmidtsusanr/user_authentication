# README

## Adding my user authentication involved the following steps:

### Controllers (app/controllers)
**ADD sessions_controller.rb**

Wondering where the log_in and log_out methods came from? Check out the controller helper.

```
class SessionsController < ApplicationController
  def new
  end

  def create
    user = User.find_by(email: params[:session][:email].downcase)
    if user && user.authenticate(params[:session][:password])
      log_in(user)
      redirect_to user
    else
      flash.now[:danger] = "Invalid email/password combination"
      render "new"
    end
  end

  def destroy
    log_out if logged_in?
    redirect_to root_url
  end

end
```

**ADD users_controller.rb**

```
class UsersController < ApplicationController
  
  def show
    @user = User.find(params[:id])
  end

  def index
  end

  def new
    @user = User.new
  end

  def edit
  end

  def create
    @user = User.new(user_params)
    if @user.save
      log_in @user
      flash[:success] = "Welcome to My DBC!"
      redirect_to @user
    else
      render "new"
    end
  end

  def update
  end

  def destroy
  end

  private

    def user_params
      params.require(:user).permit(:username, :email, :password, :password_confirmation)
    end

end
```

**ADD welcomes_controller.rb (not necessary, just a current preference choice)**

```
class WelcomesController < ApplicationController

  def index
    
  end

end
```

### Helpers (app/helpers)
**ADD sessions_helper.rb**
```
module SessionsHelper

  def log_in(user)
    session[:user_id] = user.id
  end

  def current_user
    @current_user ||= User.find_by(id: session[:user_id])
  end

  def logged_in?
    !current_user.nil?
  end

  def log_out
    session.delete(:user_id)
    @current_user = nil
  end

  def authorize
    redirect_to '/login' unless current_user
  end

end
```

### Models (app/models)
**ADD user.rb**

Wondering what has_secure_password does? The answer is a lot! Check out this 
[description on GitHub](https://github.com/rails/rails/blob/82dd60b5b7ed915dcf1eca603ea5e615c6e47a3d/activemodel/lib/active_model/secure_password.rb)

```
class User < ActiveRecord::Base
  
  has_secure_password
  
end
```

### Routes (config/routes)
**ADD the following routes**

```
Rails.application.routes.draw do
  get    'signup'  => 'users#new'
  post   'users'  => 'users#create'
  get    'login'   => 'sessions#new'
  post   'login'   => 'sessions#create'
  delete 'logout'  => 'sessions#destroy'
  resources :users
  root 'welcomes#index'
end
```

### Migrations (db/migrate)
**ADD [timestamp]_create_users.rb**
```
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :username
      t.string :email
      t.string :password_digest

      t.timestamps null: false
    end
  end
end
```

### Gem (Gemfile)
**ADD (or uncomment if it's already there):**
```
gem 'bcrypt', '~> 3.1.7'
```

### Views (app/views)
**ADD sessions folder, then ADD new.html.erb**

```
<% provide(:title, "Log in") %>
<h1>Log in</h1>

<div class="row">
  <div class="col-md-6 col-md-offset-3">
    <%= form_for(:session, url: login_path) do |f| %>

      <%= f.label :email %>
      <%= f.email_field :email, class: 'form-control' %>

      <%= f.label :password %>
      <%= f.password_field :password, class: 'form-control' %>

      <%= f.submit "Log in", class: "btn btn-primary" %>
    <% end %>

    <p>New user? <%= link_to "Sign up now!", signup_path %></p>
  </div>
</div>
```

**ADD users folder, then ADD new.html.erb**
```
<% provide(:title, 'Sign up')%>

<h2>Sign up</h2>

<%= form_for :user, url: '/users' do |f|%> 
Username: <%= f.text_field :username %><br><br>
Email: <%= f.text_field :email %><br><br>
Password: <%= f.password_field :password %><br><br>
Password Confirmation: <%= f.password_field :password_confirmation %><br><br>
<%= f.submit "Submit" %>
<%end%>
```

**ADD welcomes folder, then ADD index.html.erb**
```
<h2>Welcome!</h2>

<p><%= link_to "Log in!", login_path %></p>

<p>New user? <%= link_to "Sign up now!", signup_path %></p>
```


##### That's all you should need in order to implement user authentication! Let me know if you have any questions or feedback.
---
**Resources:**
- [Simple Authentication with Bcrypt](https://gist.github.com/thebucknerlife/10090014), by [thebucknerlife](https://gist.github.com/thebucknerlife)
- [secure_password.rb](https://github.com/rails/rails/blob/82dd60b5b7ed915dcf1eca603ea5e615c6e47a3d/activemodel/lib/active_model/secure_password.rb), [from the official Ruby on Rails GitHub page](https://github.com/rails/rails)
- [Rails Tutorial online book, chapters 5-8](https://www.railstutorial.org/book), by Michael Hartl
- [Ruby on Rails Security Guide](http://guides.rubyonrails.org/security.html)
- [Leon Gersing](https://github.com/leongersing), developer and teacher extraordinaire
