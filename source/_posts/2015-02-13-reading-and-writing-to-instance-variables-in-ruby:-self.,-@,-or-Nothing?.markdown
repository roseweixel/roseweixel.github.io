---
layout: post
title: "Reading and Writing to Instance Variables in Ruby: self., @, or Nothing?"
date: 2015-02-13 14:03:10 -0500
comments: true
categories: ['Ruby', 'object orientation', 'instance variables', 'self']
---

Today marked the end of my second week as an instructor at The Flatiron School, where I graduated just about two months ago. While working with a student this morning, I realized I had been taking for granted a piece of knowledge that I learned in week two of my own semester. 

When I first was learning about object orientation, one of the most confounding things for me was when to use @, self., or no prefix at all when dealing with instance variables in class definitions.

After several conversations with classmates and instructors, I thought I had it all straightened out. My table mates and I even wrote an 'Instance Variable Manifesto' to make sure we'd retain this enlightenment for all time. Today, I was grappling with the issue once again. It's funny how learning to program doesn't always happen in a straight line. The experience reminded me not to take for granted things I "know", and to savor the moments of re-learning.

Here's the issue that led to my temporary existential crisis today (had I totally lost my grasp of object oriented programming, all knowledge of Ruby, and/or my mind entirely?):

- We have a class, `Shoe`. 
- It has an attribute, `condition`, with an `attr_accessor`. 
- It has a method, `cobble`, that sets a `Shoe`'s condition to `"new"`.
- The Rspec test first made a new instance of `Shoe` and then set its condition to `"old"`. Then, it had the expectation that the `condition` of the shoe will be equal to `"new"` after calling the `cobble` method.

This is the code my student had:

```ruby
class Shoe
  attr_accessor :condition

  def cobble
    condition = "new"
  end
end
```

At first glance, the reason this code was not making the test pass eluded me. We put a `binding.pry` in the cobble method, and found that while `condition` was indeed set to `'new'`, the test was still failing. Wasn't `condition = "new"` just making a call to the `condition=` method provided by the attr_accessor? Apparently not. 

In fact, as I later re-realized, `condition = "new"` was creating a local variable, `condition` with the value `"new"`, which has nothing to do with the value of `@condition`. I was able to come back to understanding this concept when I recalled that I had grappled with this very same thing almost exactly four months ago, when I was working on the exact same lab. I searched through the notes I had taken during those early Flatiron days and found the "manifesto."

Four months later, here's my distilled version of it.

  - Use `@` only in the initialize method or within a custom reader/writer method (this accesses the instance variable directly)

  - If you intend to ever read or write to an instance variable, build the appropriate reader/writer methods or use `attr_accessor`, `attr_writer`, or `attr_reader` as you deem appropriate. This allows you to delegate getting and setting your instance variables to methods rather than having to manipulate the variable directly.

  - When altering the value of an instance variable, use the writer method prefixed with `self.` **Without giving the writer method an explicit receiver (`self`), the Ruby interpreter will assume you are setting a local variable rather than calling the writer method** (this is what led to the bug in the code snippet above).

  - When simply reading the value of an instance variable, the reader method may be called with no prefix. The receiver of the reader method is implicitly `self`.

Here's a code example to demonstrate usage of the previous guidelines:

```ruby
class Shoe
  attr_accessor :condition

  # we're initializing the attribute @condition, so we need to call the instance variable directly
  def initialize(condition="new")
    @condition = condition
  end

  # we're changing the value of @condition, so we call the writer method, which needs to be called on self or else Ruby thinks we're just setting a local variable
  def wear
    self.condition = "worn"
  end

  def cobble
    self.condition = "new"
  end

  # we're just reading the value of @condition, so we can call the reader method (whose implicit receiver is self)
  def report_condition
    if condition == "new"
      "This shoe is in perfect condition!"
    elsif condition == "worn"
      "This shoe needs to be cobbled!"
    else
      "The condition of this shoe is #{condition}."
    end
  end

end
```

Note that you can also use `self.` when calling reader methods, and it sometimes feels semantically more natural to do so. Unlike using `@`, this still delegates responsibility to the reader method, exactly the same as calling the reader method with no `self.` prefix.

TL;DR

- Use `@` in the initialize or reader/writer methods.

- It's better to delegate to reader and writer methods rather than accessing instance variables directly.

- You cannot call a writer method without the explicit receiver, `self`. When Ruby sees the `=` (or `-=`, or `+=`), it assumes you are setting a local variable if no receiver is specified.
