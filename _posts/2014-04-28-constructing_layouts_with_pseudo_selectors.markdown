---
title: Constructing Layouts with Pseudo Selectors
layout: post
date: 2014-04-28  9:47:54
blog: true
---

I finally got around to filling the projects page of my site with the new apps I made at [The Flatiron School](http://flatironschool.com). I had to set aside my incredible urge to just refactor and redesign the entire site in order to get the page done in a timely fashion, which reminded me of a couple of lessons I have learned while building over the past couple of months:

 - Focus on concrete features first, optimizations later.
 - Don’t let macro design considerations prevent you from getting small tasks done quickly.

Anyway, that’s not even why I’m writing this post! I wanted to talk a little bit about a cool use of CSS Pseudo Selectors that I had not considered previously: as a method of constructing alternating layouts.

## The Problem

My projects page is a series of divs, each containing an image and a short paragraph. I wanted the image and the paragraph to switch float positions on each div, alternating left-right down the page.

{% highlight html %}
<div class="project">
  <h2 class="project-title"><a href="#">Title</a></h2>
  <div class="project-content">
    <p></p>
    <a href="#"><img src="#"></a>
  </div>
</div>
<div class="project">
  <h2 class="project-title"><a href="#">Title</a></h2>
  <div class="project-content">
    <p></p>
    <a href="#"><img src="#"></a>
  </div>
</div>
<div class="project">
  <h2 class="project-title"><a href="#">Title</a></h2>
  <div class="project-content">
    <p></p>
    <a href="#"><img src="#"></a>
  </div>
</div>
{% endhighlight %}

Hard-coding the positions for each div was obviously out of the question; my CSS would've been filled with repetitive ID selectors, and it wouldn’t be at all scalable. I had to figure out a DRY solution using CSS, since I couldn't just hop into Ruby like I’m used to doing on my Rails projects (only liquid templating in Jekyll!).

## :Nth-Child

I recalled that CSS pseudo selectors could be used to style specific children in a list (li’s and p’s, mostly). For those who don't know, such selectors fall under the category of ```:nth-child(n)``` pseudo selectors, which select specific children based on what is specified by n. The most well-known values for n are “last,” “first,” “odd,” and “even.” Some common use cases for these selectors are for styling the first paragraph or word of an article in a certain manner (italicizing it, increasing its size, changing color, etc.), or for giving a striped background to tables, filling every other row with a color.

Quick side note: I learned by reviewing some documentation that you could actually select children by selector type rather than child position. Check out the example below, adapted from a [W3C wiki](https://www.w3.org/community/webed/wiki/Advanced_CSS_selectors):

{% highlight html %}
<div>
  1. <li></li>
  2. <li></li>
  3. <li></li>
  4. <li></li>
  5. <li></li>
  6. <img></img>
  7. <li></li>
  8. <li></li>
  9. <li></li>
</div>
{% endhighlight %}

Using ```li:nth-child(even)``` in this case would select number 2, 4, and 8, skipping the child in spot 6 because it is not an li. Using ```li:nth-of-type(even)``` would select numbers 2, 4, 7, and 9, counting the li’s in positions 7 and 9 as even because it pretends the img tag isn’t even there.

## The Solution

I knew that “odd” or “even” would be the best bet for my site. Looking back at the first code sample, I tried ```.project img:nth-child(even)``` and ```.project p:nth-child(even)``` in order to switch the float positions for ever other project div. This resulted in no style changes, so I tried using “1” instead of “even” to make sure I was using proper syntax. That ended up switching the float positions of all div’s on the page.

I finally realized that I wasn’t selecting what I wanted to. I wanted to change the img’s and p’s, sure, but what I was actually doing was selecting every img and p within div’s with class “project,” not every other div on the page. The reason “1” worked is because there was only one p and one img within each div. I actually needed to select each individual div and then select the elements I wanted to change. I rewrote my selectors to read ```.project:nth-child(even) img``` and ```.project:nth-child(even) p```, and I got exactly the result that I wanted!

Thus, I learned that the ```:nth-child``` CSS pseudo selector is a really effective way to define alternating patterns in the arrangement of elements. The key breakthrough for me was remembering that my project div’s were still children of body, and so I could easily use ```:nth-child``` selectors on them.

It’s easy to limit yourself to the simplest of implementation examples for patterns you haven't used extensively before: in this case, I only thought of ```:nth-child``` selectors as tools you use on elements within other div’s. In the future, I hope to short-circuit this cognitive process whenever possible and remember to think expansively of the ways in which new patterns can be applied in my code.

Now, onto completely refactoring my site!

