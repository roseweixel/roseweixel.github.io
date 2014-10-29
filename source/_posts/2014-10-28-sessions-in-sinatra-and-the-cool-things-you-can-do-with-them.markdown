---
layout: post
title: "Sessions in Sinatra (and the cool things you can do with them)"
date: 2014-10-28 23:18:15 -0400
comments: true
categories: ['Ruby', 'Sinatra', 'sessions', 'flash messages', 'search forms', 'Flatiron School']
---

Sinatra is a really neat framework for building small dynamic web apps with all the beauty of Ruby and without the bulk of Rails. Recently, my Flatiron School classmates and I built a simple playlist app that allows users to perform all the basic CRUD operations on songs - you can create, retrieve, update, and destroy them. Sinatra allows you to mimic Rails' RESTful routes for these operations by making something like this in your controller:

```ruby app/controllers/songs_controller.rb

class SongsController < ApplicationController

  get '/songs' do
  	# index action code here
  	erb :"songs/index"
  end

  get '/songs/new' do
    # new action code here
    erb :"/songs/new"
  end

  post '/songs' do
    # create action code here
    redirect :"/songs"
  end

  get '/songs/:id' do
  	# show action code here
  	erb :"songs/show"
  end

  get '/songs/:id/edit' do
    # edit action code here
    erb :"/songs/edit"
  end

  post '/songs/:id' do
    # update action code here
    redirect :"/songs/#{@song.id}"
  end

end

```

With the corresponding models and views that make these routes real, you've got yourself a working web app. Pretty cool, right?

Once I got my Sinatra app up and running with the bare minimum functionality in my controller, it was time to add some features. 

In particular, my app had two glaring problems:

1. I needed success messages that would appear right after creating or updating a song, and then disappear (the disappearing was the hard part).

2. In order to find a song, a user would have to go to the index page and browse the whole list. Even with just ninety-nine songs, this is not exactly the ideal user experience.

After much Googling and experimenting, I found a working solution to both of these probelms by using cookie-based sessions.

I used to think of "cookies" as these mysterious entities that would follow me around as I browsed. I now know that a cookie is just a small piece of text containing some information that a web site uses. Specifically, the session cookie I used to solve both of my problems is just a hash. With a little bit of `binding.pry`ing, I saw it with my own eyes:

```
{"session_id"=>"12e81670c07a19aa184f9ebc8d8d932f290267c679d9fbe5895d5297281d9404", "csrf"=>"3def15e639f06504e66e163c572b8300", "tracking"=>{"HTTP_USER_AGENT"=>"da39a3ee5e6b4b0d3255bfef95601890afd80709", "HTTP_ACCEPT_LANGUAGE"=>"da39a3ee5e6b4b0d3255bfef95601890afd80709"}}
```

Yup. That's the cookie we are referring to here as session. And what are all of those crazy strings in there? Except for the self-explanatory `:session_id`, I have no idea. What I figured out was that the usefulness of this session hash that was floating around in my program is in my ability to create, retrieve, update, and destroy things in it. I can pass these things around from one action to another and make them disappear whenever I want. Kind of like a flash message, right? 

Here's how.

First, I enabled sessions like this:

```ruby app/controllers/application_controller.rb

class ApplicationController < Sinatra::Base
  ...

  enable :sessions
end
```

Then, back in my songs controller:

  post '/songs' do
  	...

  	# After successfully creating a new song, we add a :success_message key to our session hash, with its value being the success message string.

  	session[:success_message] = "Song successfully created."
    
    redirect :"/songs"
  end



