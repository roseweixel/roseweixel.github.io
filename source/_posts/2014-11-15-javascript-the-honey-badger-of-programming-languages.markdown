---
layout: post
title: "JavaScript: The Honey Badger of Programming Languages"
date: 2014-11-15 17:49:20 -0500
comments: true
categories: ['JavaScript', 'Ruby', 'scope', 'object-oriented programming', 'arity', 'Flatiron School']
---

After spending six weeks in Ruby land, last week was all JavaScript all the time at [The Flatiron School](http://www.flatironschool.com). I feel like I've just added a hammer with superpowers to my programming tool box. You might bend a few nails along the way, but you can build almost anything with it. 

Also, JavaScript is weird. Although it cares a lot about semi-colons, there are some important ways in which, just like the honey badger, it just doesn't seem to care at all. Here I will attempt to get comfortable with this strange new animal by diving deep into some of its more surprising aspects. To provide some contrast and context, especially for those who (like myself) are more at home with Ruby, I'll make some comparisons as I go along.

## Scope

Let's dive right in. For comparison, here is some Ruby:

```ruby javascript-ruby-comparison/ruby.rb
dog = 'Fido'

def bark
  puts "#{dog} says woof"
end

bark
```

In Ruby, a method is a scope gate, so `dog` is not defined in the `bark` method. Ruby will tell you this in no uncertain terms:

```console
[18:12:24] javascript-ruby-comparison
♥ ruby ruby.rb
ruby.rb:4:in `bark': undefined local variable or method `dog' for main:Object (NameError)
	from ruby.rb:7:in `<main>'
```

On line 4, Ruby has no idea about what `dog` is, because the universe in line 4 is the local scope of the method `bark`. This scope gate can be very nice, since it means we cannot accidentally overwrite a variable we defined in the global scope from within a method. In order to make `dog` into a global variable that we can access inside the method, Ruby requires us to be very explicit about it, putting a `$` in front of that variable both where it is defined and wherever it is referenced:

```ruby javascript-ruby-comparison/ruby.rb
$dog = 'Fido'

def bark
  puts "#{$dog} says woof"
end

bark
```

Now we clearly see that we are playing with a global variable, and Ruby plays along:

```console
[18:32:01] javascript-ruby-comparison
♥ ruby ruby.rb
Fido says woof
```

While JavaScript does have a concept of functions creating a new scope, variables assigned outside of those functions can easily be accessed from within:

```javascript javascript-ruby-comparison/javascript.js
var dog = 'Fido'

function bark(){
  console.log(dog + ' says woof');
}

bark();
```

JavaScript doesn't care if you want to reach outside the function's scope to grab a variable. JavaScript functions have knowledge of the universe around them:
```console
[18:38:34] javascript-ruby-comparison
♥ jsc javascript.js 
--> Fido says woof
```

So what else can you do with a variable defined outside a function from within a function? As it turns out, JavaScript will let you do whatever you want:

```javascript javascript-ruby-comparison/javascript.js
var dog = 'Fido'

function bark(){
  dog = 'Spot'
  debug(dog + ' says woof from within the function');
}

debug(dog + ' says woof outside of the function');
bark();
debug(dog + ' says woof after the function is called');
```

What happened to Fido?

```console
[20:47:14] javascript-ruby-comparison
♥ jsc javascript.js 
--> Fido says woof outside of the function
--> Spot says woof from within the function
--> Spot says woof after the function is called
```

Looks like the name change was permanent. And by the way, by always using `var`, we can prevent such dangers:

```javascript javascript-ruby-comparison/javascript.js
var dog = 'Fido'

function bark(){
  var dog = 'Spot'
  debug(dog + ' says woof from within the function');
}

debug(dog + ' says woof outside of the function');
bark();
debug(dog + ' says woof after the function is called');
```

```console
♥ jsc javascript.js 
--> Fido says woof outside of the function
--> Spot says woof from within the function
--> Fido says woof after the function is called
```

By using `var` within the function, we created an entirely different `dog` and did not alter the original.

## Arity Schmarity

Before we go any further, I must admit that hadn't heard the word arity until I crossed paths with JavaScript. I reckon I'm not alone in this, so here's a definition, courtesy of [Wikipedia](http://en.wikipedia.org/wiki/Arity):

> In logic, mathematics, and computer science, the **arity** of a function or operation is the number of arguments or operands the function or operation accepts.

OK. This sounds like something I know from Ruby land. Here's a familiar example:

```ruby javascript-ruby-comparison/ruby.rb
class Dog
  def initialize(name)
    @name = name
  end

end
```

According to the above definition, the `initialize` method of the `Dog` class takes exactly one argument. You could say its arity is one (more commonly, this is known as a unary function).

Ruby cares about arity, and will give you very nice error messages to make you aware of this:

```console
2.1.2 :002 > require 'ruby.rb'
 => true 
2.1.2 :003 > fido = Dog.new
ArgumentError: wrong number of arguments (0 for 1)
	from /Users/rose/Development/code/javascript-ruby-comparison/ruby.rb:3:in `initialize'
	from (irb):3:in `new'
	from (irb):3
	from /Users/rose/.rvm/rubies/ruby-2.1.2/bin/irb:11:in `<main>'
```

I tried to make a new `Dog` with 0 arguments, and Ruby was expecting 1. So Ruby blew up. This is arity checking at work.

JavaScript does not care about arity! Here's photographic proof (this is a lot easier to see in the browser's JavaScript console):

{% img center /images/js-no-arity-checking.png 700 700 'image' 'images' %}

To review what just happened: 

* I made a constructor function for `Dog` (yeah, functions are really all-purpose in JavaScript, so that right there was a class definition of sorts).
* In this constructor function, I said that `Dog` takes a `name`.
* I made a new `Dog`, `fido`, and did not pass in any arguments. 
* JavaScript did not blow up.
* Just to be sure, I asked JavaScript to show me `fido`.
* JavaScript returned a `Dog` object with a `name` of `undefined`.

Whenever it can, JavaScript will try to help you out, whether you want it to or not. If you say a `Dog` has a `name` and then forget to give it one, JavaScript will trust that this was your intent and will throw in an `undefined` rather than yelling at you.

## Teaching Old Dogs New Tricks

Ruby cares about defining an object's attributes and behaviors up front. Objects in Ruby behave in consistent and predicatable ways. 

Using our prior definition of the `Dog` class, Ruby will let us know that we haven't taught `Dog`s to bark yet:


```console
2.1.2 :001 > require 'ruby.rb'
 => true 
2.1.2 :002 > fido = Dog.new('Fido')
 => #<Dog:0x007f9fd5757c78 @name="Fido"> 
2.1.2 :003 > fido.bark
NoMethodError: undefined method `bark' for #<Dog:0x007f9fd5757c78 @name="Fido">
	from (irb):3
	from /Users/rose/.rvm/rubies/ruby-2.1.2/bin/irb:11:in `<main>'
```

Of course! We did not define a `bark` method, so we get a predictable `NoMethodError`.

JavaScript doesn't care about defining all of an object's attributes and behaviors up front, and pretty much lets you do whatever you want, whenever you want. Here's a continuation of the console session above:

{% img center /images/js-fido-barks.png 700 700 'image' 'images' %}

* I asked `fido` to bark, having never defined a `bark` function or attribute. JavaScript quietly returned `undefined`.
* I told JavaScript to give `fido.bark` the value of `'woof'`, and JavaScript happily obliged.
* Now not only can `fido` bark, but the `Dog` object that is `fido` has been fundamentally altered, now containing a new `bark` attribute!

So, can all `Dog`s bark now? 

{% img center /images/js-spot-cant-bark.png 700 700 'image' 'images' %}

No, they cannot. Now the definition of `Dog` is a bit of a mess - `fido` is a `Dog` that can bark, while `spot` can't. The very nature of `Dog`-ness has been brought into question, and JavaScript is fine with all of this. No errors, no complaining. For better or for worse.

## Playing with Global Scope

This one is extra weird, so get ready.

In the hierarchy of Ruby objects, `main` is the top level:

```console
♥ irb
2.1.2 :001 > self
 => main 
2.1.2 :002 > self.class
 => Object 
2.1.2 :003 > 
```

I don't know why you'd want to mess with `main`, and Ruby makes certain it's not easy to do so (in fact, it's impossible to do so by accident):

```console
2.1.2 :003 > self.name
NoMethodError: undefined method `name' for main:Object
	from (irb):3
	from /Users/rose/.rvm/rubies/ruby-2.1.2/bin/irb:11:in `<main>'
2.1.2 :004 > self.name = "Rose"
NoMethodError: undefined method `name=' for main:Object
	from (irb):4
	from /Users/rose/.rvm/rubies/ruby-2.1.2/bin/irb:11:in `<main>'
2.1.2 :005 > 
```

Want to mess with the top-level object in JavaScript? Go right ahead, JavaScript doesn't care!

```console
>>> this.name
undefined
>>> this.name = "Rose"
Rose
>>> this
[object global]
>>> this.name
Rose
>>> 
```

I just named the global object after myself, and JavaScript did not care one bit. This example is a bit contrived, but believe me that some undesireable things can happen if you don't watch out for global assignment. Here's a (slightly) less contrived example:

```javascript javascript-ruby-comparison/javascript.js
function Dog(name){
    this.name = name;
    this.hunger = true;
    this.bowl = 'full';
}

Dog.prototype.checkNeeds = function() {
    if(this.hunger === true){
        (function feed(){
            this.hunger = false;
            this.bowl = 'empty';
        })();
    } else {
        (function play(){
            this.hunger = true;
        })();
    }
};

var fido = new Dog('Fido');

debug(fido.hunger);

fido.checkNeeds();

debug(fido.hunger);
debug(this.hunger);
debug(this);
```

If I run this code, here's what my `debug` statements reveal:

```console
♥ jsc javascript.js 
--> true			// fido was born hungry
--> true			// after calling checkNeeds(), he's still hungry...
--> false			// ...but this isn't hungry...
--> [object global]	// because we fed the global object instead of fido!
```

As you can see, Bad Things can happen when the global object is always available. Fortunately, there is a way for you to make JavaScript care about this! You just have to tell JavaScript to `use strict`.

```javascript javascript-ruby-comparison/javascript.js
'use strict';

function Dog(name){
	// the rest of my code...
```

Now, when we run the code, JavaScript will blow up just the way we want it to!

```console
♥ jsc javascript.js 
--> true
Exception: TypeError: Attempted to assign to readonly property.
feed@javascript.js:12:17
checkNeeds@javascript.js:14:11
global code@javascript.js:24:16
```

## The Moral of the Story

This was a long post, and it probably could get longer. But I think this is enough to chew on for now. The bottom line is that JavaScript has some crazy ways, but these ways are not unlearnable, and the reward for learning them is a much saner coding experience. If you understand the quirks, remember your `var`s, and always `use strict`, you and JavaScript can be unstoppable, just like the honey badger.

{% img center /images/nothing-stops-honey-badger.png 500 500 'image' 'images' %}
