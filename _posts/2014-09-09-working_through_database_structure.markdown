---
title: Working Through Database Structure
layout: post
date: 2014-09-12 8:40:00
blog: true
---

So remember when I said that I had figured out the schema for my new app idea and just had to implement it? Well, that was kind of a lie.

I've got most of the base models down: Movie, Actor, Book, Author, TVShow, etc. These are pretty straightforward, and I've built apps that have revolved around these types before.

I was worried for a little bit about how I was going to handle Genre—would I have to distinguish between a MovieGenre and a BookGenre? But I realized that genres are just dumb tags, so all of the media types in the app would be able to use the same Genre type. I can just build the app so that you can filter by genre only when you're looking at a specific list type: your book list or movie list as opposed to your global list. That maps onto typical user behavior pretty well, and I can always change it later.

The trickier part of my schema is the concept of "category." I want to let users save actors, directors, or movies to their lists because that's what I like to do. Usually I want to check out a particular movie, but sometimes I have heard of a particular director, and rather than specify which movie I want to see of theirs I would like to just jot the name down and deal with it later.

To account for this flexbility, my app will know that actors, directors, and movies all belong to the same category: "film," let's say. That way, users could visually spot film items on their global list and filter down to their film list to see just those items. I prefer this approach to letting users create and manage their own categories because I would rather this structured organiztion be built into the app's identity and design.

I will use duck-typing to my advantage here, giving every media model in my app a category method that returns a category construct. But what is that construct?

A symbol like :film would be super simple, but hard-coded things suck if I feel like changing the category name later, particularly if it's written into each model. To mitigate that problem, I could define the symbols in a module like Categorizable, but then I have an unwiedly case statement checking for types that I might have to add to over time. It's doable, but not ideal.

What if Category were a class? I would get the benefit of encapsulating the name so that it only needs to be changed in one place, and I would just associate each category with the right models. I could also implement something like ```Category#items``` to return media saved by users under that category, which sounds like a nice API.

However, I'm worried that I would have to define an instance's category each time it is saved, for every single media type. I would have a ton of category= statements throughout my API service classes without a ton of benefit. That could be remedied by ```after_create``` hooks within the model, but that feels like a lot of repetition and overhead for what is effectively something that will be hardcoded and most likely left alone. With a whole new model, I would also have a lot of database records and extra Ruby baggage just to encapsulate a string and a method that could easily be implemented differently.

When I began this blog post, I was pretty convinced that I would end up implementing categories as a class. Surely defining everything as objects is inevitable, and I would just have to write enough words to convince myself of how stupid it would be to forgoe objects for primitives.

But you know what? I think I might actually go ahead with the hard-coded symbols after all, at least for now. Yes, it's pretty repetetive to write them into each media type model. But I don't really anticipate changing the category names, and it's pretty easy to find and replace my way through that problem if I come to it. Also, I would have to change the names in the front-end anyway, and I don't think creating a Category class magically solves that issue.

Bascially, why should I go crazy trying to pre-optimize a feature that really can be implemented this simply? I could be totally wrong and end up building a Category class down the road. But if that happens, at least I would be doing it because I had to solve an actual problem rather than just assuaging a vague fear that [Uncle Bob](http://en.wikipedia.org/wiki/Robert_Cecil_Martin) will come yell at me or something.

