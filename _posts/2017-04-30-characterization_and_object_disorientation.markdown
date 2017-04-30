---
layout: post
title:  "Characterization and Object Disorientation"
date:   2017-04-30 21:57:40 +0000
---


One of the challenges of learning to code (for me, anyway) has been integrating the design principles I've learned along the way into new contexts. Object orientation is case in point. Back in January, when I was working my way through the Ruby lessons, I felt like I had a pretty good handle on -- encapsulation, separation of concerns, each object being responsible for its own data. But as I sat down to refactor my Rails portfolio project, I realized that those principles weren't being fully implemented in my current work. So I went back and re-read my notes from those early classes, and re-watched some of the lectures that seemed especially pertinent. 

One particularly crystalizing concept that I came across in those lectures is: *objects should be the main character of their own methods.*  This came up in the context of another clarifying concept: *Users shouldn't be God objects.* The basic idea is this: if you have a blogging app where Users can sign up and create Posts, and you want to write a method that retrieves all the posts from a particular user -- e.g. `#show_all_posts` -- which object should contain that method -- `User` or `Post`? The intuitive answer, at least for me, is `User`. 

But this is what was meant by "Users shouldn't be God objects". Given the principle that each object should be responsible for its own data, should a User really be answering questions about a `Post`? In one sense, *everything* in a domain like this is related to the User -- that's the whole point of building the app in the first place. But the *User object* should only be responsible for managing and manipulating the attributes of a given `User`, and maintaining the inventory of the `User` class as a whole.  `Post`, on the other hand, should know about all of its instances, and the Users that created them. 

Reflecting on this particular piece of wisdom led me to a design flaw in my project. It's a user interface for a real brick-and-mortar library, where library Patrons can browse and check out books, and Librarians can manage loans, add to and edit the collection, and so on. ("Librarian" and "Patron" are enumerated `roles` of my domain's `User` model). When a Patron checks out a book, the title and due date are displayed on his or her `#show` page: 

<a href="http://imgur.com/z6euz7f"><img src="http://i.imgur.com/z6euz7f.png" title="source: imgur.com" /></a>

Behind the scenes, this display is generated with a whole mess of code that starts like this: 

```
<ul>
   <%@user.checked_out_books.each do |l|%>
     <%if l.overdue?%>
       <li><%=l.book.title%> ||
       OVERDUE <%if current_user.patron?%> -- PLEASE RETURN <%end%>
```

And yes, that's already a lot of logic for a View -- I refactored later with helpers. But after thinking about characters and god-objects, what really seemed out of place was this method: `@user.checked_out_books`. As you can see, that's a `User` method, build into the model like so: 

```
 `def number_of_loans
  	  checked_out_books.size
  	end
  
  	def checked_out_books
   		loans.checked_out.map {|l| l.book}
  	end`
```

And that right there is what you're *not* supposed to do in a good object-oriented program. The question that's being asked there -- "what books have been checked out by this user?" -- isn't a question about Users. It's not even a question about Books. It's a question about *Loans* -- a third model, which acts as a join between `Book` and `User`. So the solution is to refactor these methods into the `Loan` model: 

```
def self.number_of_loans(patron)
	  checked_out_books(patron).size
	end
	
  
  def self.checked_out_books(patron)
	  patron_loans = includes(:patron).where(patron_id: patron.id)
	  patron_loans.checked_out.map {|l| l.book}
	end
```

...which can then be called as: `Loan.checked_out_books(@user)`.

One refactor down... many, many dozens more to go....
		
		

