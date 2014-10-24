---
layout: post
title: "Join Tables in ActiveRecord: Complex Associations, Simple Code"
date: 2014-10-22 21:38:48 -0400
comments: true
categories: ['Ruby', 'Active Record', 'data modeling', 'television', 'The Flatiron School']
---

Up until a few weeks ago, my entire coding toolbox consisted of a small sample of some built in Ruby methods. Two of these became blog post topics ([#find](http://roseweixel.github.io/blog/2014/10/08/looking-for-something-just-number-find-it/) and [#slice](http://roseweixel.github.io/blog/2014/10/07/cheese-dot-slice/)), and one---the `<<` method---became the central character of my first coding dream/nightmare, with reserved words `while` and `true` playing key supporting roles. I used these methods to write other pretty simple methods. 

After just a few weeks at [The Flatiron School](http://www.flatironschool.com), I'm quickly learning that building up complex programs, like web apps, can't (or maybe really just shouldn't) be accomplished by writing out method after method in one big file. You need the right structures and tools to create complex objects, persist them in a database, and express relationships between these objects. Using ActiveRecord, you get some really simple and powerful tools for all this that hide a huge amount of complexity away, allowing you focus on designing your app to do what you want it to do. 

While ActiveRecord makes it easy to create models and associations, some of the concepts involved were hard for me to get my head around at first. One such concept was the join table. I ran up against a problem in need of a join table while working with my classmates on a lab involving modeling characters and actors. 

In ActiveRecord speak, actors have many characters and a character belongs to an actor. Here, in all of its elegant simplicity, is how you'd create the models that express this relationship:

```ruby app/models/actor.rb
class Actor < ActiveRecord::Base
  has_many :characters
end
```
```ruby app/models/character.rb
class Character < ActiveRecord::Base
  belongs_to :actor
end
```
Now all we need are two tables (actors and characters) to make this association work. The characters table, being on the `belongs_to` side of the relationship, must have a column for the foreign key, `actor_id`. In ActiveRecord, this can be accomplished in remarkably few lines of code:

```ruby db/migrations/001_create_actors.rb
class CreateActors < ActiveRecord::Migration
  def change
    create_table :actors do |t|
      # ActiveRecord provides an id (primary key) column by default, for free!
      t.string :name
    end
  end
end
```

```ruby db/migrations/002_create_characters.rb
class CreateCharacters < ActiveRecord::Migration
  def change
    create_table :characters do |t|
      t.string :name
      # Here is where the magic happens. ActiveRecord knows that this references the actors table.
      t.integer :actor_id
    end
  end
end
```

This is the basic pattern you'd follow to model any `has_many` `belongs_to` relationship in ActiveRecord. What seems like "magic" here is made possible by ActiveRecord Ruby methods that either give our associated classes more methods that let them interact with each other, or wrap SQL statements and hide them away (in the case of migrations). But that is a topic for another post.

Back to actors and characters. As my classmates and I were creating these models, two big things came to the surface: 

1) We could not recall any of Tom Cruise's many characters' names.

2) What happens when a character has more than one actor? One of my favorite characters would most certainly break our has many/belongs to association---The Doctor.

{% img center /images/sad_doc_rain.jpeg 500 500 'image' 'images' %}

How can we fit The Doctor into our current schema??? The more I thought about it, the more impossible it seemed. Here is an illustration:

{% img center /images/actor_char_table.png 500 500 'image' 'images' %}

The above is our simple model, before The Doctor comes along and breaks everything. Just to be sure I was grasping how NOT to try to include The Doctor and his many actors, here is how it might look:

{% img center /images/the_doctor_broke_the_table.png 800 800 'image' 'images' %}

Our actors table seems fine, but our characters table is definitely not okay, and we haven't even included the eight doctors from the first twenty-six seasons yet. 

With our current setup, we have to change the entire schema every time we add another Doctor. This is very, very bad. A database was designed for adding lots of rows. Not so for columns.

In order to make things right, we need a different association: many-to-many. Actors have many characters, and a character has many actors. Thus, we would replace `belongs_to :actor` with `has_many :actors` in `app/models/character.rb`. But how do we get our database set up for this more complicated association?  We need a join table. First the visual:

{% img center /images/actors_characters_join.png 650 650 'image' 'images' %}

Now for the migration (assuming your actors table is still intact and you've already removed all but the name column from characters):

```ruby 004_create_actors_characters_join_table.rb
class CreateActorsCharactersJoinTable < ActiveRecord::Migration
  def change
    create_table :actors_characters, id: false do |t|
      # using id: false prevents the default primary key column from being created, because there's no use for it here
      t.integer :actor_id
      t.integer :character_id
    end
  end
end
```

Now we have the right associations and the right database schema to add all of the Doctors that ever were and ever will be throughout all time.

{% img center /images/first_doctor.jpeg 500 500 'image' 'images' %}