---
layout: post
title: "Looking for something? Just #find it!"
date: 2014-10-08 20:32:04 -0400
comments: true
categories: ['Ruby', '#find', 'food']
---

Searching through nested data structures can be downright messy. Even saying "nested data structures" sounds complicated. Luckily, Ruby has some powerful [Enumerable methods](http://www.ruby-doc.org/core-2.1.3/Enumerable.html) that can simplify our lives (or at least our code).

Let's say you've been programming for hours and suddenly realize your stomach is growling at you (why does this happen so often?). Here is how you might search for some delicious food, if you happened to be searching inside an array filled with hashes. Just humor me for the sake of example.

Here is our `restaurants` array:

```ruby find_delicious_food.rb
restaurants = [
	{:name => "The Decent Diner", :rating => "average"}, 
	{:name => "The GoGo Grill", :rating => "delicious"}, 
	{:name => "Emporium of Mystery Meat", :rating => "poor"},  
	{:name => "Dig Inn Seasonal Market", :rating => "delicious"}
]
```
So, we want to get the name of a restaurant whose `:rating` is `"delicious"`. We could do something like this:

```ruby
def find_delicious_food(restaurants)
	  restaurants.each do |restaurant_info|
    restaurant_info.values.each do |value|
      if value == "delicious"
        return restaurant_info[:name]
      end
    end
  end
end
```

If we call this method on our `restaurants` hash, it will return the value "The GoGo Grill". Sounds good to me. But in order to get there we needed a `return` inside an `if` statement inside an `each` loop inside another `each` loop. Not pretty.

There is a better way. The `find` method! Behold:

```ruby
def find_delicious_food(restaurants)
  restaurants.find{|restaurant| restaurant[:rating] == "delicious"}[:name]
end
```

Just one line of code seems to do the same thing as its ugly predecessor. Let's look at it more closely. Here is what `find` does:

* It passes each element of the object on which it was called to a block.
* It returns the first element for which the block evaluates to `true`.
* If none of the elements return true for the given block, it returns `nil`.

As it turns out, this `nil` value could cause some problems for our `find_delicious_food` method. If there are no restaurants in the hash with a `:rating` of `"delicious"`, our code breaks. We would get an error, because Ruby cannot make any sense out of `nil[:name]`.

Here is the method refactored to avoid that error:

```ruby
def find_delicious_food(restaurants)
  good_restaurant = restaurants.find{|restaurant| restaurant[:rating] == "delicious"}
  if good_restaurant
    return good_restaurant[:name]
  end
end
```

Sure, it may not be as cute and little as our one-line method, but it will never break and it's still a lot prettier than loops inside loops. 

So when you are searching for that *one* thing you need inside of some nasty nesting, just `find` it!
