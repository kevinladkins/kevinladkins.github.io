---
layout: post
title:  "The Joy of jQuery"
date:   2017-06-13 16:54:00 +0000
---


One rewarding aspect of learning new coding dialects is discovering solutions to nagging problems. 

The original version of my Rails portfolio project had an issue that I couldn't find a solution to. The app is a library interface -- essentially a web portal like you'd find for your local public library, where patrons can browse the collection and check books in and out. Users registered as Librarians can also add to and edit the collection. When a new book is added, the Librarian has the option of choosing an existing author from the database or adding a new one; of assocaiting the title with one or more existing categories; and of creating a new category. 

This led to an extremely unweildy form, with fields for every possible input: 


<a href="http://imgur.com/Of5R8hz"><img src="http://i.imgur.com/Of5R8hz.png" title="source: imgur.com" /></a>


It's not the *worst* form ever created, but it's awfully busy and hard get your head around. More importantly, the sub-forms -- New Author and Category -- are only *sometimes* needed. If the book's author is already in the database, or the librarian chooses not to add a category, those fields remain blank. I remember thinking at the time: *Gee, it sure would be nice if there were some way to only show those fields if they're needed* -- e.g. to toggle them on and off via a link or a button. But I couldn't see any way to do that using Ruby (short of reloading the page entirely, which didn't seem worth it). 

With jQuery, however, it's a snap. 

There's really next to nothing to it. All it required was adding a couple of links to the HTML view...

```
<a href="#" id="add-new-author-button">Add New Author</a> || <%end%>
<a href="#" id="select-categories-button">Select Categories</a>
```

...wrapping the sub-forms in `<div>s` with their own ids...

```
<div id ="new-author-field-set">
<div id="select-categories-field-set">

```


...then, over in my book.js file, adding event listeners to the links...

```
$("#add-new-author-button").click(function() {
     displayAuthorFieldSet();
  });
  $("#select-categories-button").click(function() {
     displayCategoryFieldSet();
  });
```

...which trigger the jQuery `toggle()` function when clicked.

```
function displayAuthorFieldSet() {
  $("#new-author-field-set").toggle()
};

function displayCategoryFieldSet(e) {
    $("#select-categories-field-set").toggle()
};
```

And that's it! Here's the much more concise, revised form layout:

<a href="http://imgur.com/KFPWjP5"><img src="http://i.imgur.com/KFPWjP5.png" title="source: imgur.com" /></a>

It's a silly little thing, but there's something immensely satisfying in discovering that there's a simple, ready-made solution to a previously confounding issue. 



