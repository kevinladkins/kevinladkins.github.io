---
layout: post
title:  "That "Keyser Soze" Moment"
date:   2017-03-06 19:24:10 -0500
---


I think it was about two weeks ago that I hit the lesson "<a href = "https://learn.co/tracks/full-stack-web-dev-with-react/sinatra/mvc-and-forms/video-review-forms">Video Review: Forms</a>." I'll be honest -- I generally skip the "video review" lessons in favor of moving on to the next instructional unit. But in this case -- seeing as how I'd just blown through SQL, ActiveRecord, and Rack in the space of a week and half and wasn't entirely sure I'd even retained any of *that*, let alone HTML material from two months previous -- I decided to invest the hour or so it would take for the review. That hour or so ended up being more like three hours, as it turned out that the "Forms Review" actually introduced a whole swath of new subjects, including RESTful design and the Sinatra Request Cycle. 

It was somewhere between the bit on the Request Cycle and the explanation of how forms are posted to Controllers that I had what I'd call a "<a href = "https://en.wikipedia.org/wiki/The_Usual_Suspects">Keyser Soze </a> moment." 

All at once, all those seemingly disperate subjects I'd been struggling through for the past months -- Ruby, Object-Oriented Design, databases, web protocols etc. -- coalesced into a single, unified picture: the "full stack" of Web development. There's something incomparably thrilling about that moment when all the pieces come together. Suddenly seeing how Ruby classes interact with ActiveRecord databases via Sinatra Controllers to display web pages in HTML -- it made everything that had come before seem worth it. 

So when it came time to do my Sinatra portfolio project, I found myself legitimately *excited* to put this newfound understanding to work. To put the cherry on top, my chosen project also allowed me to indulge my insufferably Hipsterish devotion to vinyl records: 

<a href="http://imgur.com/eXRbjd0"><img src="http://i.imgur.com/eXRbjd0.png?1" title="source: imgur.com" /></a>

It's a pretty straightforward app for cataloging a record collection. Users can add and edit records from their home (Index) page, and view their collection by Record, Artist, or Label:

<a href="http://imgur.com/KnIna8O"><img src="http://i.imgur.com/KnIna8O.png?1" title="source: imgur.com" /></a>

Those last two views entailed probably the trickiest coding in terms of database queries. It's not hard to list the Records, where you can just call the User object with `<%@user.records.order(name: :asc).uniq.each do |r|%>` and then output each title onto a table with `<%@user.records.order(name: :asc).uniq.each do |r|%>`. But because I'd set the app up in such a way that, in addition to each user's collection, there's also a master database containing *all* records added by *all* users, extracting only the titles by a particular artist or label that are also part of *one particular user's* collection threw me for quite a loop initially: 

<a href="http://imgur.com/VRT8HLP"><img src="http://i.imgur.com/VRT8HLP.png?1" title="source: imgur.com" /></a>

The solution is actually pretty simple, and presumably obvious to anyone who's been doing this for any length of time. Because Artists and Labels are associated with Users via a *has many, through: :records* relationship, all you have to do is iterate through the Artist objects stored in the user's `artists` (or `labels`) array (`<%@user.artists.order(name: :asc).uniq.each do |a|%>)`, compare each record's foreign key (which establishes their *belongs to* relationship with Artists and Labels) with the id of each Artist or Label object (`<%@user.records.where("artist_id == #{a.id}").order(name: :asc).each do |r|%>`) , and then output the record's `name` attribute (`<%=r.name%>`). Simple, in retrospect; at the time, considerably less so. 

Of course, as is my wont, I also spent unseemly amounts of time tinkering around with features that weren't even part of the assignment (see also <a href = "https://kevinladkins.github.io/2017/02/09/what_doesnt_kill_me/">this</a>). For example, I thought it would be nice to build a little banner up top with some navigational links... 

<a href="http://imgur.com/lR8sjos"><img src="http://i.imgur.com/lR8sjos.png?2" title="source: imgur.com" /></a>

...which ended up requiring many hours of Dante-like descent into the unholy hell that is HTML element positioning (I've lost whole days trying to definitively wrap my head around `position: relative` vs `position: absolute` vs `position fixed` and how they interact with `float`, to no discernable avail). And after all that, what I ended up with looks.... okay. Not bad for an amateur, perhaps, but not exactly professional quality. That's a ways off yet, but still -- these last couple of weeks have felt like serious steps forward. 

