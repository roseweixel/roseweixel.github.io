---
layout: post
title: "cheese.slice"
date: 2014-10-07 22:07:16 -0400
comments: true
categories: ['Ruby', '#slice', 'food']
---

When learning a programming language as natural as Ruby, the syntax inevitably seeps into our thoughts and alters the way we conceptualize everything from the complex to the mundane. Thus, the title of this blog (and, indeed, this post).

It is easy for this to happen, in part because Ruby has such an extensive library of methods whose names and behaviors make them ideal for modeling real life. One such method is `slice`.

The method `slice` can be called on an array (or an object that acts like an array, like a string). It literally returns a "slice" of the object on which it is called. The original object is left intact.

	> cheese = ['gouda', 'muenster', 'provolone', 'manchego', 'brie']
	> my_slices = cheese.slice(1, 2)
	=> ["muenster", "provolone"] 
	> cheese
 	=> ["gouda", "muenster", "provolone", "manchego", "brie"]

If you need to take more drastic measures, `slice!` will modify the original object, removing (and returning) everything you sliced out.

	> cheese = ['gouda', 'muenster', 'provolone', 'manchego', 'brie']
	> my_slices = cheese.slice!(1, 2)
	=> ["muenster", "provolone"] 
	> cheese
 	=> ["gouda", "manchego", "brie"]

Here are three different ways to use `slice` (`slice!` can also be invoked in these ways, and both can be used with arrays and strings):
	
  * `array.slice(index)` returns the object at `array[index]`, or `nil` if there is no object there to return.

  * `array.slice(start, length)` returns a new array containing the elements of `array` starting at `start` and continuing for `length` elements (as shown in the code examples above). If there are no objects there, it returns `nil`.[^fn-one]

  * `array.slice(range)` returns a new array containing the objects at `array[range]`, or `nil`.

  For more on this and other Ruby Array methods, see the [documentation](http://www.ruby-doc.org/core-2.1.3/Array.html#method-i-slice).

  [^fn-one]: There are some special cases that return an empty array. See the [documentation](http://www.ruby-doc.org/core-2.1.3/Array.html#method-i-slice).


