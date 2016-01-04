# Setting up ActionMailer with Gmail

Overall, setting up ActionMailer within your rails app is pretty straightforward. However, if you're just starting out like I am, there are few small steps that can easily cause some missteps. Hopefully this tutorial will help you avoid making the same mistakes I did!

This tutorial assumes you know how to make a basic rails app with full CRUD functionality.

- Basic setup
- Passing instance variables to your mailer
- Common errors

## Step 1: Updating config/environments/development.rb


For some reason rails turns error messages off for its mailer, so they're first thing we're going to do is turn that on. Find the code below in the development.rb file and set it to true

```rb
config.action_mailer.raise_delivery_errors = false
```

```rb
config.action_mailer.raise_delivery_errors = true
```

Next we'll add in your delivery method using Gmail. Add the following (you can add it anywhere, but I put it under the code we just set to true so that it's all together)

```rb
config.action_mailer.delivery_method = :smtp

config.action_mailer.smtp_settings = {
address: "smtp.gmail.com",
port: 587,
domain: ENV["GMAIL_DOMAIN"],
authentication: "plain",
enable_starttls_auto: true,
user_name: ENV["GMAIL_USERNAME"],
password: ENV["GMAIL_PASSWORD"]
}
```

It's important to note the use of ```ENV[""]``` here. This is one step we take towards hiding sensitive information that we don't want posted to github for anyone to see. More on this later.

If you're using devise the following code should be at the bottom of development.rb file, but if not make sure you add it.

```rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

## Step 2: Updating config/environments/production.rb

Having mailers work in development is great, but let's quickly update our app so it's prepared for production as well(This assumes you're using)

Add the following code to your production.rb file. Make sure to update the host with your actual heroku app's url

```rb
config.action_mailer.default_url_options = { host: 'http://yoursite.herokuapp.com' }

config.action_mailer.delivery_method = :smtp

config.action_mailer.smtp_settings = {
address: "smtp.gmail.com",
port: 587,
domain: ENV["GMAIL_DOMAIN"],
authentication: "plain",
enable_starttls_auto: true,
user_name: ENV["GMAIL_USERNAME"],
password: ENV["GMAIL_PASSWORD"]
}
```

## Step 3: Creating our mailer

Now for the fun part, actually creating the mailer. In your command line enter the following. The last option for the command can be whatever you want. I chose welcome_user.

```
$ rails generate mailer UserMailer welcome_user
```

You'll notice this creates a number of files for you.

```
create  app/mailers/user_mailer.rb
create  app/mailers/application_mailer.rb
invoke  erb
create    app/views/user_mailer
create    app/views/layouts/mailer.text.erb
create    app/views/layouts/mailer.html.erb
create    app/views/user_mailer/welcome_user.text.erb
create    app/views/user_mailer/welcome_user.html.erb
invoke  test_unit
create    test/mailers/user_mailer_test.rb
create    test/mailers/previews/user_mailer_preview.rb
```

The first file we're going to modify out of these is app/mailers/user_mailer.rb. This assumes you have a user model already.

Let's edit it to look like this
```rb
class UserMailer < ApplicationMailer
  def order_ahead_email(user)
    @user = user
    mail to: user.email, subject: "Welcome"
  end
end
```

This looks and behaves very much like a controller. You can see that we pass a in a user to the function and then set it as an instance variable so we can then use it in our views. We'll pass the actual user in our controller. I usually pass in more than just the user so that I can easily access them in the views as well.

Now we'll want to invoke the mailer action in our controller. Here I do it in the User controller, but you can place it wherever you want depending on when you want an email to be sent out.

```rb
class UsersController < ApplicationController
  def create
    @user = User.new(params[:user])
    UserMailer.welcome_user(@user).deliver_now
  end
end
```

Next we get to customize the views. Some people only receive text files so it's good practice to make text emails, but I'll only be focusing on html emails

Short and sweet email:
```html
<p>
  <%=@user.name %>, thank you for joining our site!
</p>
```

## Step 4: Using our sensitive information safely

First let's get your mailer to work in development

Add this gem:

```rb
gem 'figaro'
```

In your command line
```
$ bundle install
$ figaro install # creates config/application.yml
```
In your newly created config/application.yml file add

```rb
GMAIL_DOMAIN:gmail.com
GMAIL_USERNAME:your user name here
GMAIL_PASSWORD:your password here
```

Congrats, you did it!! Test it out by creating a user.

Lastly, to make it work in heroku go to your app's settings and under config variables enter in the same keys and values that are in your application.yml file.

Enjoy your new mailer :)
