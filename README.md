# Setting up ActionMailer with Gmail

Overall setting up ActionMailer within your rails app is pretty straightforward. However, if you're just starting out like I am, there are few small steps that can easily get you caught up. Hopefully this tutorial will help you avoid making the same mistakes I did!

- Basic setup
- Passing instance variables to your mailer
- Common errors

## Step 1: Updating config/environments/development.rb


For some reason rails turns error messages off for its mailer, so they're first thing we're going to do is turn that on.

```rb
config.action_mailer.raise_delivery_errors = true
```
