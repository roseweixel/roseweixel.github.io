---
layout: post
title: "Live Updating DOM Elements with jQuery and Ajax
date: 2015-06-04 21:20:11 -0400
comments: true
categories: ['Ajax', 'jQuery', 'JavaScript', 'Rails', 'live updates']
---

Any web app that involves real-time interactions between users requires some form of live notifications. Implementing this, for a beginner developer such as myself, can be a daunting challenge. This post will walk through how I went about solving this problem when working on [Lacquer Love&Lend](http://www.lacquerlove.com), a social network for nail polish lovers that allows users to interact via friendships and lacquer loans. As with any social network, I wanted users to see live notifications whenever they received a new friendship or transaction request, or when the `state` of any of their friendships or transactions changed. The example that follows assumes some basic knowledge of Rails.

## Some Basic Ajax

In a basic Ajax request, a user clicks on something, the Ajax request gets sent, and a part of the DOM gets updated without the entire page reloading. 

{% img center /images/basic_ajax_request.png 830 830 'image' 'images' %}

The code usually looks something like this:

```javascript app/assets/javascripts/something.js
// 1) Wait for the document to be ready.
$(document).ready(function() {
    // 2) Listen for the submission of the form.
    $("form").submit(function(event) {

        // 3) Prevent an entire page load (or reload).
        event.preventDefault();

        // 4) Grab the information from the form needed for the Ajax request.
        var formAction = $(this).attr('action'); // e.g. '/somethings'
        var formMethod = $(this).attr('method'); // e.g. 'post'
        var formData   = $(this).serializeArray(); // grabs the form data and makes your params nicely structured!

        // 5) Make the Ajax request, which will hit the 'create' action in the 'somethings' controller
        $.ajax({
          url:  formAction,
          type: formMethod,
          data: formData
        });
    });
});
```

For the basic example, I'm omitting the controller and view as the main focus of this post is how I implemented live notifications. A more detailed explanation of basic Ajax will follow in my next post - a prequel to this one, if you will :).

With the code above, a single user's action of submitting the form sets off the whole chain of events. But for live notifications, more than one user is involved and the action that changes one user's data is hapenning on another user's client! Making this happen twisted my brain into a pretzel at first, but after several attempts I got the functionality I wanted. A description of these follows below.

To Be Continued...
