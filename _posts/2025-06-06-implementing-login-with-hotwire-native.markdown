---
layout: post
title: "Implementing login with Hotwire Native and Ruby on Rails"
date: 2025-06-06 07:36:00 +0200
categories: open-source
---

## Implementing login with Hotwire Native and Ruby on Rails

It is a very common behaviour for mobile and web applications to have different layouts for unauthenticated and authenticated users. In this post, we will create a Ruby on Rails application with Bootstrap 5.3, scaffold user resources, add devise for authentication, and implement a bottom navigation bar. Then we will create an Android application and add a native bottom navigation bar!

## Setting up a new Rails application

Create a new Ruby on Rails application using the Bootstrap frontend framework.

```sh
rails new hotwire-login-web --css bootstrap
```

Star the rails server.

```
bin/dev
```

Open [localhost:3000](http://localhost:3000), you should see the Rails welcome page.

![welcom-rails](/images/implementing-login-with-hotwire-native/1-rails-welcome.png)

Yaay, we are on Rails!

## Scaffolding some views to work with

Let's generate our one and only user resources.

```sh
rails g scaffold user
```

Then set the route route to `users#idex`.

```rb
# config/routes.rb
root "users#index"
```

Open [localhost:3000](http://localhost:3000), you should see the index page for users.

![welcom-rails](/images/implementing-login-with-hotwire-native/2-devise-login-page.png)

## Setting up devise

Add the devise gem to your application.

```sh
bundle add devise
```

Install devise.

```sh
rails generate devise:install
```

After installation devise will list some manual configuration steps, let's go over them!

Make sure that `action_mailer.default_url_options` are set in for development environement:

```rb
# config/environment/development.rb
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

Add flash messages for the applciation layout.

```erb
# app/views/layouts/application.html.erb
<body>
    <p class="notice"><%= notice %></p> <!-- add this line -->
    <p class="alert"><%= alert %></p> <!-- and this -->
    <%= yield %>
</body>
```

Generate User model with devise.

```rb
rails generate devise user
```

Run the migrations.

```rb
rails db:migrate
```

Open [localhost:3000/users/sign_in](http://localhost:3000/users/sign_in), you should see the devise login page.

![devise-login](/images/implementing-login-with-hotwire-native/2-devise-login-page.png)



