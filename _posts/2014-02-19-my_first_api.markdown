---
title: My First API
layout: post
date: 2014-02-19 18:05:46
blog: true
excerpt: "My dad and I were always interested in extracting meaning from Rotten Tomatoes' scores. One night, after watching an amazing foreign thriller, we were curious: did the general populace think this movie was as good as we thought it was?"
---

My dad and I have always obsessively tried to extract meaning from Rotten Tomatoes' movie scores. One night, after watching an amazing foreign thriller, we were curious: did the general populace think this movie was as good as we thought it was? Could you conclude anything about the movie in question based on the difference between the critic score and the user score? The thriller, it turns out, got a score of 12, meaning the critics percentage was 12 higher than users.

From that moment on, a little game was born whenever we watched a movie together: when you subtracted the users score from the critics score, what was the result?

 - Transformers: Revenge of the Fallen (-38)
 - Gravity (13)
 - Dallas Buyers Club (2)
 - Jackass: The Movie (-28)
 - The Artist (10)

Even after searching for some popular movies, I began to find what seemed to be patterns. Popular action or comedy movies would have a fairly significant "negative" filmetric leaning toward users, whereas foreign films or "indie" movies would often have a high "positive" filmetric. In fact, when I told a friend this, he suggested I call the calculation a "snob score." I decided on "filmetric."

At the time, I knew that it was possible to acquire this data and perform calculations on the movie scores that I found. I even started to think about the possibilities afforded by computing filmetrics—recommendations to users, averages by genre or by director, a fun guessing game, perhaps. Yet these dreams seemed far off given that I didn't think I would ever become a programmer. Amazing what can change in just a few months time!

Once we began web scraping during our Flatiron labs, I remembered my filmetric idea and decided to google around. Lo and behold, Rotten Tomatoes had an API! It hadn't been updated for about a year, but people seemed to still be using it. I was very intimidated by the prospect of applying for an API key, but I knew I had to try it anyway.

As it turns out, applying is not actually that onerous a process, at least from my experience. In fact, "applying" is quite the overstatement for what actually transpired. They for a summary of my application idea, an application URL, and some other info, but I'm pretty sure I could've written absolutely anything. They approved me within 2 minutes of submitting the form, making it highly doubtful that anyone read it. So if you're scared about applying for an API key like I was, don't be—it probably won't be as big a deal as you think.

With API key in hand, I looked up the [documentation](http://developer.rottentomatoes.com/docs/read/Home) for a basic RT JSON query, loaded up IRB, and tried it out!

{% highlight ruby %}
  require 'open-uri'
  html = open(
  'http://api.rottentomatoes.com/api/public/v1.0/
  movies.json?apikey=[xxhfj689]...')
  => OpenURI::HTTPError: 403 Forbidden
{% endhighlight %}

Well, that didn't work! Pro tip: when the API Doc uses brackets where they expect you to insert your API key, they generally don't actually want you to type them. Let's try again!

{% highlight ruby %}
  html = open(url)
  #<StringIO:0x00000102049270 ...>
{% endhighlight %}

Obviously, this wasn't what I wanted either. Loading the url in my browser worked fine, but IRB wasn't actually returning a JQuery string. After reading a few helpful StackOverflow articles, I was finally able to convert the JSON into a messy hash.

{% highlight ruby %}
  response = Net::HTTP.get_response(URI.parse(url))
  json = response.body
  result = JSON.parse(json)
{% endhighlight %}

I could then pull some useful data out by drilling down in the hash, just like we've learned!

{% highlight ruby %}
  result["movies"][0]["title"] => "Toy Story 3"
  result["movies"][0]["ratings"]["critics_score"] => 99
  result["movies"][0]["ratings"]["audience_score"] => 89
  result["movies"][0]["critics_consensus"] =>
  "Deftly blending comedy, adventure, and honest emotion,
    Toy Story 3 is a rare second sequel that really works."
{% endhighlight %}

I was happy to find that I could easily get the two pieces of data I was most interested in (the audience and critics scores) for any movie I wanted, including a lot more data that could prove useful, including actors, movie rating, and runtime.

I found this method of parsing data rather inelegant, and I wasn't looking forward to constantly constructing URLs in Ruby. I decided to see if anyone happened to have written a gem to levearage this API in a cleaner fashion. Since this is the Ruby community after all, two people had done just that! I picked the best [one](https://github.com/nmunson/rottentomatoes) and played around with it.

{% highlight ruby %}
  require 'rottentomatoes'
  include RottenTomatoes

  movie = RottenMovie.find(:title => "The Lion King")
  movie.title => "The Lion King"
  movie.ratings.critics_score => 90
  movie.ratings.audience_score => 93
  movies.critics_consensus =>
  "Emotionally stirring, richly drawn, and beautifully
  animated, The Lion King stands tall within Disney's
  pantheon of classic family films."
{% endhighlight %}

Now that I know I can retrieve useful data with ease, a few questions remain. What is the best way to store a lot of movies from Rotten Tomatoes in a local databse using these API calls? How much information should I keep concerning each movie to allow for flexibility later on? What kind of interfaces should I use to best present my filmetrics within a useful context?

I won't be able to answer all of these questions until I keep working with the data, but I do think one way to proceed would be to write a script to automate API calls while remaining within the limits set by the API (5 calls per second, 10,000 per day). This would require me to either have a list of thousands of movie titles to iterate over, or some way to sucessfully generate random Rotten Tomatoes id's and use them to query the site. The API allows for querying based on other criteria as well, so maybe some other method would work even better. I also will need to build an ORM or utilize a preexisting one such as Sequel to store all of the data I pull.

If I've learned anything from this exercise, it's that the Flatiron School has already made me extrememly well-equipped to build compelling applications of this data. I can't wait to get started!
