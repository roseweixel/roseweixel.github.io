---
layout: post
title: "Flash Messages with Sessions in Sinatra"
date: 2014-10-28 23:18:15 -0400
comments: true
categories: ['Ruby', 'Sinatra', 'sessions', 'flash messages', 'Flatiron School']
---

[Sinatra](http://www.sinatrarb.com/) is a DSL (domain specific language) for building small dynamic web apps with all the beauty of Ruby and without the bulk of Rails. Recently, my [Flatiron School](http://www.flatironschool.com) classmates and I built a simple playlist app that allows users to perform all the basic CRUD operations on songs - you can create, retrieve, update, and destroy them. Emulating [RESTful routes for these actions (à la Rails)](http://guides.rubyonrails.org/routing.html#crud-verbs-and-actions), I created something like the following:

```ruby app/controllers/songs_controller.rb

class SongsController < ApplicationController

  get '/songs' do
    @songs = Song.all
    erb :"songs/index"
  end

  get '/songs/new' do
    erb :"songs/new"
  end

  post '/songs' do
    @song = Song.create(params[:song])
    redirect :"/songs/#{@song.id}"
  end

  get '/songs/:id' do
    @song = Song.find(params[:id])
    erb :"songs/show"
  end

  get '/songs/:id/edit' do
    @song = Song.find(params[:id])
    erb :"songs/edit"
  end

  post '/songs/:id' do
    @song = Song.find(params[:id])
    @song.update(params[:song])
    redirect :"/songs/#{@song.id}"
  end

end

```

With the corresponding models and views that make these routes real, a working web app was born. Pretty darn cool. But there was a feature in the spec that was still left to add, and it became a bit of a challenge:

Whenever a song is created or updated, the user should see a message like this:

{% img center /images/song_success_message.png 700 700 'image' 'images' %}

Here's what I wanted to achieve:

- The user fills out the form at 'songs/new' and clicks 'Submit'.
- The newly created song's show page 'songs/:id/show' is rendered with a success message.
- The user reloads the page (or navagates away and back), and \*poof\* the success message is gone.

In order to get this to work, I would have to somehow pass a message from one action in the controller to another. The create (or update)action would have to pass a message along to the show action which would pass this message along to the view. It would be nice to have something like an envelope called `success_message` that could hold one of three values at any given time:

- `"Successfully created song."` (right after the song is created)
- `"Successfully updated song."` (right after the song is updated)
- `nil` (immediately after the success message is rendered in the view)

<a name="erb"></a>
According to this vision, I was able to put some yet-to-be-functional code in `songs/show.erb`:

```erb
<!-- this bit of code at the bottom of the page would reveal the contents of the success message if it isn't nil -->
<% if @success_message %>
  <div id="success" style="border: 2px solid green;">
  <%= @success_message %>
  </div>
<% end %>
```

But how would `@success_message`, which would have to be definied in the show action in order for the show page to know about it, be able to hold a message the was created in another controller action (create or update)?

I knew that `params` is one such data object that holds values that can be passed from one action to another, but as it is already the envelope for carrying and delivering form data entered by the user, it didn't seem like the right place for a success message to live. If only there was some other vessel, like params, but not...

<a name="first_step"></a>
There is. It's a session! It becomes available to you as soon as you put this in your application controller:

```ruby
class ApplicationController < Sinatra::Base
  ...
  # add this line to enable the magic
  enable :sessions
end
```

Once you've added that, you get this handy little object called `session` that you can access in any of your controllers, which can in turn pass along any value it contains to a view.  Just to see what it looked like before doing anything with it, I put a `binding.pry` inside my index action in `songs_controller.rb`, fired off `shotgun`, and loaded the index page on my local server. Here's what pry spat out when I asked it what `session` was:

```irb
[1] pry(#<SongsController>)> session
=> {"session_id"=>"857d0d856ee646d76204c1138cf57fc497f80ffd85fef19befdbb26f60f8e022", "csrf"=>"598119612cef0d88134413ddd54bad52", "tracking"=>{"HTTP_USER_AGENT"=>"7be1a42d74a413474898ddb9adfef9a5a84719e3", "HTTP_ACCEPT_LANGUAGE"=>"66eae971492938c2dcc2fb1ddc8d7ec3196037da"}}
```
Here it is again, split up onto several lines for readability:

```ruby
{
  "session_id"=>"857d0d856ee646d76204c1138cf57fc497f80ffd85fef19befdbb26f60f8e022",
  "csrf"=>"598119612cef0d88134413ddd54bad52",
  "tracking"=>{
      "HTTP_USER_AGENT"=>"7be1a42d74a413474898ddb9adfef9a5a84719e3",
      "HTTP_ACCEPT_LANGUAGE"=>"66eae971492938c2dcc2fb1ddc8d7ec3196037da"
  }
}
```

So that's our session object, before we've done anything with it. Inside it there's `"session_id"`, a unique key that lives inside your cookie for the life of your session. What else is in there? Having just learned about preventing cross-site request forgery, I presumed `"csrf"=>"598119612cef0d88134413ddd54bad52"` to be the CSRF token. If you want to learn more about this (and you should!), read about it [here](http://guides.rubyonrails.org/security.html). Finally there's `"tracking"`, which I believe holds information about the user's browser. For our current purposes (creating a flash message), we don't need to care about these values, but it helps to see how session is structured and what kinds of things it can contain.

Let's move on and see how we can use this session object to help us with our success message.

Before doing anything to it, it would help to understand what kind of object `session` is. It looks like an ordinary Ruby hash, but in fact it is a special kind of Rack object. Why guess when you can `pry` it?

```irb
[2] pry(#<SongsController>)> session.class
=> Rack::Session::Abstract::SessionHash
```

As it turns out, the `SessionHash` object has a lot in common with a `Hash` object. 103 methods in common (out of 177 total), to be exact:

```irb
[3] pry(#<SongsController>)> session.methods.select{ |method| Hash.methods.include?(method) }.count
=> 103
```

One `Hash`-like thing you can do with `session` is put new keys into it, with values that you can assign and change at your whim. And since the session persists from one HTTP request to another, it can pass values around from one action in the controller to another. This sounded like just the right object to hold the success message!

Here's what I did (after enabling sessions, as shown [above](#first_step)):

1) At the bottom of my create action (immediately before redirecting to the show page), I added a `:success_message` key to `session`, setting its value to the one-time message I want my users to see:

```ruby songs_controller.rb
  post '/songs' do
    @song = Song.create(params[:song])

    session[:success_message] = "Successfully created song."

    redirect :"/songs/#{@song.id}"
  end
```

2) I did the same thing for my update action:

```ruby
  post '/songs/:id' do
    @song = Song.find(params[:id])
    @song.update(params[:song])

    session[:success_message] = "Song successfully updated."

    redirect :"/songs/#{@song.id}"
  end
```

3) Then, I added some code at the top of the show action to pass this message along so it could be rendered by the [erb template](#erb) in the view, and to ensure that the message is immediately set to `nil` so it will only render once:

```ruby
  get '/songs/:id' do
    @song = Song.find(params[:id])
    
    @success_message = session[:success_message]
    session[:success_message] = nil

    erb :'songs/show'
  end
```

Now my success message behaves exactly as I imaged it should, appearing and disappearing at the appropriate times.

Due to the stateless nature of HTTP requests, had I simply used an instance variable `@success_message` inside these `get` and `post` actions, any value assigned to it would have been instantly forgotten the moment another request was made. The `@success_message` inside the two actions would have in fact been completely out of each other's scope. The session, however, is available across all actions.

This ability to fake statefulness in your controllers opens up a world of functionality other than flash messages, not the least of which is maintaining a user's "logged in" state as they browse from one page to another. Without sessions, almost all of the interactions and transactions that occur over the Internet as we know it could not even happen. 

So if you haven't yet, sit down and get to know the powerful little session. It's definitely worth prying into.

