---
layout: post
title:  "what_doesnt_kill_me ==...*"
date:   2017-02-08 21:18:56 -0500
---





# ** ..."makes me stronger" || "drives me inexorably toward madness" ?**

One of the themes that's developed as I work my way through the Flatiron program is: the hard stuff's never what you think it's gonna be. Labs that I expect to be grueling slogs breeze by in under and hour; labs that look like quick, uncomplicated exercises turn into daylong plummets through multidimensional portals of recursive error. 

I'm about a quarter of the way through the program now, and I've just wrapped up my first Final Project. The assignment was simple enough: build a CLI interface that gathers data from the web and allows users to access additional information based on their selections. The program I decided to build is *top-movies-all-time*, a basic CLI app that lists the top-grossing films of all time -- broken down by Domestic, Inflation-Adjusted, and Worldwide box office -- and then displays more detailed breakdowns of individual titles based on user selections. The design is similarly straightforward -- a ::CLI class that interfaces with the user, a ::Scraper class that gathers data from www.boxofficemojo.com, and a ::Movie class that generates objects for each title on the lists. 

Now, let's pause for a moment to reflect on this line from the assignment page, styled in bold so we won't miss it: 

**Please note that while you will be writing code to scrape data from a public website, what we're really looking for is your capacity to effectively write good object oriented ruby code (objects, not hashes, separation of concerns, etc.) - we're less interested in the details of the regex's or selector you're using to parse the web pages that you're scraping.**

Pretty clear, yeah? If you're gonna spend an inordinate amount of time on something, it should be the objects, not the web-scraping.

So of course, I ended up spending two solid days on web-scraping. 

Here's what happened.

If you go to Box Office Mojo and look up their All Time Box Office pages, what you'll see is this:

![](http://i.imgur.com/vM45JUM.png)

Nothing fancy, just a ranking of films by total gross, with each title linking to a dedicated page with a more detailed breakdown on each movie. To gather this info, I used the Nokogiri gem -- it's a nifty little gem that was the subject of a couple of those labs I thought would take forever but ended up being a cinch. I figured all I'd need to do was go to the pages (there's one each for Domestic, Adjusted, and Worldwide), open up the Inspector, and then spend an hour or so refining my CSS selectors with Nokogiri to target the data I needed. 

I was so, so wrong. 

Box Office Mojo has been around forever, well before the days of what they used to call "Web 2.0", and it hasn't changed much since its inception. This is evident not only in its layout -- which is pretty old-school by today's standards -- but in its architecture. The site's structure was clearly built well before HTML5, as evidenced by its use of outdated tags like `<b>` and `<i>`. This also means that it's lacking in all those "semantically meaningful" tags like `<header>` and `<article>` that don't actually affect the display but make it easier to locate and sift through content. And if you've ever wondered how important "semantic meaningfulness" really is to a page's design, take a look at what happens when you try to figure out where those individual movie titles on Box Office Mojo actually "live":

![](http://i.imgur.com/Ltew6it.png)

To emphasize: that's a string of text located at the rock bottom of a table cell, inside a hyperlink, inside `<b>` tags, *nested inside like six other tables*, with nary a class-identifier or ID tag in sight. 

Kids: don't build your websites like this. It's mean. It's thoughtless. It makes people think you don't care. And although you may not realize it, it's probably making some poor schlub out in New Jersey literally -- yes, *literally* -- pull his hair out while sputtering incoherent obscenities at his computer. 

I'm not going to go into detail on the literally (okay, *figuratively*) infinite number of `"table tbody tr table[2] tr[3] tr[4] table tr[2]"`-type CSS combinations I cycled through before *finally* figuring out how to reliably hone in on the data I was after -- unlike the architects of boxofficemojo.com, I'm not a cruel man -- but suffice to say, it was not quick. When I was done, the magic formulas turned out to be: 

```
html.css("div#main div#body table table tr td[2] a b").text 
```
...for titles,

```
html.css("div#main div#body table table tr td[1]").text 
```
...for rankings, and...

```
html.css("div#main div#body table table tr td[2] a btd[2] a").attribute("href").value
```
...for URLs. 

So -- all in all, an infuriating task, but a finished one. Now, about those well-designed objects that the bold text made such a deal of... 

Well, to start with, you've gotta grab the HTML and serve it up to Nokogiri. My app is built around content from three separate web pages, so my initial stab at it went like this...

```
def get_domestic_list
   Nokogiri::HTML(open("http://www.boxofficemojo.com/alltime/domestic.htm"))
end
 
def get_worldwide_list
   Nokogiri::HTML(open("http://www.boxofficemojo.com/alltime/world/"))
 end
 
 def get_adjusted_list
   Nokogiri::HTML(open("http://www.boxofficemojo.com/alltime/adjusted.htm"))
 end
```

...which is, yes, not great design. You've got three separate functions getting called independently to do the same basic thing, i.e. serve up a big messy glob of HTML node sets. Worse, there's about a half dozen other functions using that data, and if you have to call them each three times, it's gonna get ugly. Ahh, but that's what hashes are for, right? So how about: 

```
  def self.get_lists
    lists = {}
    lists[:domestic] = Nokogiri::HTML(open("http://www.boxofficemojo.com/alltime/domestic.htm"))
    lists[:adjusted] = Nokogiri::HTML(open("http://www.boxofficemojo.com/alltime/adjusted.htm"))
    lists[:worldwide] = Nokogiri::HTML(open("http://www.boxofficemojo.com/alltime/world/"))
    lists
  end
```

Much better. From there, I built a method for drilling down to the "meat' of each movie's table row: 

```
def self.scrape_list(list)
    list.css("div#main div#body table table tr")
  end
```

....which I then used to create a nice, compact method that parses the data from the lists and returns a hash with each movie's rank and title as key/value pairs:

```
def self.rankings
  rankings = {}
  scrape_list(get_lists).each {|t| adjusted_rankings[t.css("td[1]").text] = t.css("td[2] a b").text}
  rankings
end
```

Except... 

Did I say "reliably hone in on" earlier? Well -- not quite. For reasons known only to God and the designers of boxofficemojo.com, the first couple table cells at those CSS locations contain a big ugly mess of data that looks a bit like this: 

```
"Jump to #1\u0096100Jump to #101\u0096200Jump to #201\u0096300Jump to #301\u0096400Jump to #401\u0096500Jump to #501\u009
6600Jump to #601\u0096700Jump to #701\u0096800Jump to #801\u0096900Jump to #901\u00961000Jump to #1001\u00961100Jump to #1101
\u00961200Jump to #1201\u00961300Jump to #1301\u00961400Jump to #1401\u00961500Jump to #1501\u00961600Jump to #1601\u00961700
Jump to #1701\u00961800Jump to #1801\u00961900Jump to #1901\u00962000Jump to #2001\u00962100Jump to #2101\u00962200Jump to #2
201\u00962300Jump to #2301\u00962400Jump to #2401\u00962500Jump to #2501\u00962600Jump to #2601\u00962700Jump to #2701\u00962
800Jump to #2801\u00962900Jump t
```

...only *way* longer. So... fine. Just "shift" off the first couple cells, and you're good, yes? Well, *sort of* -- because, just to make things even more infuriating, it's a *different number* of cells depending on which list you're dealing with. So instead of just one nice, abstracted #rankings method, you're forced to use *three*, not-at-all-abstracted methods: 

```
def self.adjusted_rankings
    adjusted_rankings = {}
    scrape_list(get_lists[:adjusted]).each {|t| adjusted_rankings[t.css("td[1]").text] = t.css("td[2] a b").text}
    adjusted_rankings.shift
    adjusted_rankings
  end

  def self.domestic_rankings
    domestic_rankings = {}
    scrape_list(get_lists[:domestic]).each {|t| domestic_rankings[t.css("td[1]").text] = t.css("td[2] a b").text}
    5.times {domestic_rankings.shift}
    domestic_rankings
  end
```
...etcetera.

But whatever. It gets the job done, and it could be worse. So -- onto the next task. My program's design calls for the scraped data to be chopped into individual chunks representing each movie on the list and then passed through to the ::Movie class to instantiate new objects with :title and :url arguments. There's a bit of a problem with that URL bit, though, because -- wouldn't you know it -- Box Office Mojo's links are inconsistent; sometimes they  link directly to the movie's page, and other times they link to a landing page that requires a clickthrough. To deal with this problem, I created another method:

```
def self.url_normalizer(url)
    url.scan(/page=releases/) == [] ? "http://www.boxofficemojo.com#{url}" : "http://www.boxofficemojo.com#{url.split("releases").join("main")}"
  end
```

So far, so good. Then we've got to extract the url and title...

```
def self.get_title(list)
    list.css("td[2] a b").text
  end

  def self.get_url(list)
    url_normalizer(list.css("td[2] a").attribute("href").value)
  end
```


...and finally, pass that data to the ::Movie object:

```
def self.make_movies
    scrape_list(get_lists).each do |k, v|
      v.each {|t| TopMoviesAllTime::Movie.create_from_list(get_title(t), get_url(t))}
    end
  end
```

So -- does it work? Have I, despite a gusher of messy and uncooperative data from a maddeningly uncooperative website, managed to create a pretty succinct and efficient little function for generating my ::Movie objects?

Of course not! What that lovely little block of code will get you is:

```
/home/kevinladkins-39364/code/examples/top-movies-all-time-cli-app/lib/top_movies_all_time/scraper.rb:12:in `scrape_list': un
defined method `css' for #<Hash:0x000000012ccc88> (NoMethodError)   
```

I won't go into the ins and outs of it -- partly out of pity for you, but mostly because my brain is starting to melt just thinking it all through again -- but the issue boiled down to this:

Way up at the start, when I built my #get_lists method to deal with the three lists in a single method, I created a hash with `:domestic`,` :adjusted`, and `:worldwide` as keys, and their associated HTML nodesets as values. This works perfectly fine if you access them individually, as in `get_lists[:keyword]`, but if you try to run them with `.each` (or `.collect`, or `.map`... believe me, I tried them all), the return value ends up being: `nil`. And of course, if you try to run .css or any other method on `nil`, you get: `(NoMethodError) `.  Now, is there a way to deal with this issue? Perhaps. But whetever it is, I couldn't find it -- I `Pry`ed, I `irb`ed, I Googled... no luck. So -- back to the drawing board once again. 


And that's just the start of the fun! But this blog is veering quickly into David-Foster-Wallaceian bulk, so I'm gonna leave it at that. Overall, the strategy I ended up with was quarantining the ugly, Rube-Goldbergish code-parsing functions into the ::Scraper class so my ::CLI and ::MOVIE classes could stay clean and abstract like good little objects should. That was my "process", and it resulted in a program that... is as objected-oriented as I could figure out how to make it. Let's hope the bold text is okay with that. 

-------
&#42; *While we're on the subject, I should add that I got about three-quarters of the way through writing this blog when I accidentally hit the "back" button on my browser -- which, despite the ubiquitous "Saved" notice that pops up every five seconds in the "Draft" status in the corner, resulted in the COMPLETE LOSS OF EVERYTHING I'D WRITTEN TO THAT POINT. I don't want to belabor the point, but that was... not cool. No, it was not cool at all.*


