---
layout: post
title: "Making Friends (With Code)"
date: 2014-12-04 23:01:41 -0500
comments: true
categories: ['Rails', 'ActiveRecord', 'The Flatiron School']
---

This post will not be about how I've spent the last ten weeks at [The Flatiron School](http://flatironschool.com/) making awesome friends while coding together, or about my process of becoming friends with code. Those are posts for another time. This one is about the kinds of friendships that are actually built with code, the kind you can persist in a database.

Over the course of working on my two most recent projects, I've discovered that there is more than one way to make a friendship happen on the backend of a Rails app (or most likely any app). My first crack at making friendships was for a [nail polish sharing app](https://lacquer-love-and-lend.herokuapp.com/) where friends can view each other's collections and ask to borrow from each other. I watched Treehouse's tutorial on [building social features in Ruby on Rails](http://teamtreehouse.com/library/building-social-features-in-ruby-on-rails), which is a good place to start if you've never done something like this before. Although it helped me understand it somewhat, the way it was done in the tutorial still seemed pretty complicated to me. I decided to try to figure it out on my own so that I could break it down and understand it better.

Here are the basic back-end requirements for creating a friendship (clearly some buttons and forms on the front-end are needed, but those are implementation details, so I'll leave them out of this post):

* You have to teach your models that some Users have relationships with other Users (a self-referential association).

* You have to store that relationship in the database so that both sides of the relationship have knowledge of it and access to it.

* You have to have a way of representing and keeping track of the state of the relationship at any given time (requested, accepted, etc.)

Here's how I did it in my first go-around:

1) I created a join table, `friendships`:

```ruby
class CreateFriendships < ActiveRecord::Migration
  def change
    create_table :friendships do |t|
      t.integer :user_id
      t.integer :friend_id
      t.timestamps
    end
    # it's a good idea to add an index for faster look-ups in the database
    add_index :friendships, [:user_id, :friend_id]
  end
end
```

2) In the model for Friendship, I taught it that Friends are really just other Users:

```ruby
class Friendship < ActiveRecord::Base
  belongs_to :user
  belongs_to :friend, class_name: 'User', foreign_key: 'friend_id'

  validate :is_not_duplicate, :on => :create

  def is_not_duplicate
    if !(Friendship.where(:friend_id => friend_id, :user_id => user_id).empty? && Friendship.where(:user_id => friend_id, :friend_id => user_id).empty?)
      errors.add(:friendship, "This friendship already exists!")
    end
  end
end
```

As you can see from the above, I decided I didn't want to allow "duplicates" of friendships in the database, but as it turns out there are reasons why you might not want to do it this way. I'll go into that later.

3) I made the User model aware of the Friendship model:

```ruby
class User < ActiveRecord::Base
  has_many :friendships
  has_many :friends, through: :friendships

  validates :name, presence: true, uniqueness: true

  def accepted_friends
    accepted_friends = []
    Friendship.where(user_id: self.id, state: 'accepted').each do |friendship|
      accepted_friends << User.find(friendship.friend_id)
    end
    Friendship.where(friend_id: self.id, state: 'accepted').each do |friendship|
      accepted_friends << User.find(friendship.user_id)
    end
    accepted_friends
  end

  def requested_friends_awaiting_approval
    pending_friends = []
    Friendship.where(user_id: self.id, state: 'pending').each do |friendship|
      pending_friends << User.find(friendship.friend_id)
    end
    pending_friends
  end

  def friendships_for_your_approval
    pending_friendships = []
    Friendship.where(friend_id: self.id, state: 'pending').each do |friendship|
      pending_friendships << friendship
    end
    pending_friendships
  end

  def friends_for_your_approval
    pending_friends = []
    Friendship.where(friend_id: self.id, state: 'pending').each do |friendship|
      friend = User.find(friendship.user_id)
      pending_friends << friend
    end
    pending_friends
  end

  def all_friends
    accepted_friends + requested_friends_awaiting_approval + friends_for_your_approval
  end

end
```

You'll notice that I wrote a whole bunch of methods to tell Users how to find their friends on both sides of the relationship. I wanted to give users custom notifications in the views depending on whether they had a pending request or a request awaiting their approval, which was part of the reason for this. But even if I hadn't wanted to do that, I would have at least needed some kind of setup like this that makes multiple hits to the database every time someone wants to see all of their friends.

4) Finally, `state` is a big part of this model. I set it up so a friendship could have a state of `pending` or `accepted` (similar methods would create other states, like `rejected`). This is where the friendships controller comes in. For the sake of brevity, I'll just describe the `create` and `update` methods:

```ruby
class FriendshipsController < ApplicationController

  def create
    @user = current_user
    if params[:friendship] && params[:friendship].has_key?(:friend_id)
      @friend = User.find(params[:friendship][:friend_id])
      @friendship = Friendship.new(friend: @friend, user: @user, state: 'pending')
      @friendship.save
      flash[:notice] = "Your friendship with #{@friend.name} is pending."
      redirect_to(:back)
    else
      flash[:notice] = "Friend required"
      redirect_to root_path
    end
  end

  def update
    @friendship = Friendship.find(params[:id])
    @friendship.update(state: params[:state])
    redirect_to(:back)
  end

end
```

Since I've made a column in the friendships table for `state`, I can set and update the value of that attribute so a Friendship can introspect on the state it is currently in (what if we could write a method for doing this in real life?).

I set the `state` of the friendship to `'pending'` whenever it first gets created. Then, when the User who was "friended" decides to hit the 'Accept' button (or the 'Reject' button as the case may be), the friendship can get updated accordingly.

So, although the implementation of friendships I ended up with in my first attempt has the functionality I wanted, but as I mentioned before, it's not the most efficient. While my reasoning was that it is not desirable to have two rows in the friendships join table for every relationship that gets created, this reasoning did not take into account the way databases work. Because I have to query twice (Who are the people I've friended? Who are the people who have friended me?) every single time I want to access my friends, this is actually unneccesarily expensive. Since rows in a database are cheap, why worry about adding an extra row if it reduces the number of times you need to hit the database?

I got a second crack at this problem during my current project, an app for teachers that includes functionality for generating seating charts according to who works well together and who doesn't. Along with my teammates, we built Buddyships (and Enemyships, too!) in a more reciprocal way:

```ruby
class Enemyship < ActiveRecord::Base
  belongs_to :student
  belongs_to :enemy, class_name: 'Student', foreign_key: 'enemy_id'
  belongs_to :course_section

  after_create :inversify
  after_destroy :destroy_inverse

  validates_uniqueness_of :student, :scope => [:enemy, :course_section_id]
  validates_uniqueness_of :enemy, :scope => [:student, :course_section_id]
  
  validate :cant_be_own_enemy, :cant_be_buddies_and_enemies

  def inversify
    self.enemy.enemyships.create(:course_section_id => self.course_section_id, :enemy_id => self.student.id)
  end

  def destroy_inverse
    if inverse = self.enemy.enemyships.where(:enemy_id => self.student.id, :course_section_id => self.course_section_id).first
      inverse.destroy
    end
  end

  private
  def cant_be_own_enemy
    if student_id == enemy_id
      errors.add(:enemy_id, "you can't be your own enemy! :(")
    end
  end

  def cant_be_buddies_and_enemies
    if Buddyship.find_by(student_id: student_id, buddy_id: enemy_id, course_section_id: course_section_id)
      errors.add(:enemy_id, "you can't be both buddies and enemies for the same class! :(")
    end
  end
  
end
```

In this implementation, two rows in the join table are created for each relationship, using a method aptly called `inversify`.

Because of the reciprocal nature of the relationship created in this model, one database query (`student.enemyships`) is all it takes to find all potential Enemies. If you were only creating relationships, you'd care about not writing to the database more than you needed to. But since, in general, we query for friendships way more frequently than we create them, we want to make sure the querying is as efficient as possible.

So, while one-way relationships are fine for a follower-followee type of relationship, if you want to create a reciprocal relationship in your next app, don't shy away from those extra rows in the database.
