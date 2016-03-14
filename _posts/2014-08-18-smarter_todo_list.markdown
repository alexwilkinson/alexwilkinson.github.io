---
title: Building a Smarter ToDo List
layout: post
date: 2014-08-18
blog: true
---

I've started a new project recently, which I'm really excited about. I unfortunately did so right when I began a new job, so I won't have a lot of time to devote to it. But it's refreshing to be thinking about something other than the Rails projects I began at The Flatiron School. Don't get me wrong—I love tweaking my existing code and adding on features to bring my ideas closer to their full potential—but it was time to move on.

Starting with a fresh Rails directory is pretty exciting, and I'm looking forward to putting into place new best practices. Testing as I go will be a big one, as well as mocking external API requests from the very beginning. I'm also going to try to reduce my dependency on factories, since I won't always need to persist data in order to test (see this article from [Thoughtbot](http://robots.thoughtbot.com/speed-up-tests-by-selectively-avoiding-factory-girl)). I also generated my rails project using The Flatiron School's Rails application [generator](https://github.com/flatiron-school/flatiron-rails), which has greatly reduced the time I waste deciding which tools to start with and actually get to the business of building the application.

## The Basic Idea

For years I've been keeping lists of movies, books, and music that I want to check out. I've used a variety of apps to accomplish this, and have recently settled on [Vesper](http://vesperapp.co/) as my note application of choice. I love Vesper, but like all plain text note-taking systems, it cannot store meaningful metadata on the items I'm recording. I'm tired of manually entering URLs and Rotten Tomatoes scores and Ratings in order to provide meaningful context. I'm also tired of hopping among multiple, highly-specific apps just to save media for later. I love focused app design, but I hate switching to a movie list app to save movies and the Amazon shopping app to save books and the Rdio app to save artists.

I know there has to be a better solution, and I'm done waiting around for someone else to solve this problem for me. So, I've decided to build a smarter media list app that I'm calling Immedialist (immediate + media + list! cute, right?). The app will parse any title you enter -- whether it be a song, book, or movie -- and grab important meta data automatically: reviews, associated people, download links, or whatever else. I've wireframed the layout, researched the APIs I'll need, and mapped out a basic database structure. I'll use Rails for the back-end and a JavaScript framework for the front-end: probably Ember because it looks fun to use and hooks in so well with Rails APIs, but Angular is a contender as well.

## Next Steps

We'll see what happens when I start building Immedialist in earnest, but I think it's my most compelling idea. Even if it doesn't end up being that great, working on this will level up my UI/UX knowledge and allow me to play around more with JavaScript frameworks, which is awesome. The biggest challenge will be finding the free time to tinker with Immedialist while working as a full-time web developer, but that's something I'll have to figure out whether I want to or not. Side projects are an increasingly expected aspect of a productive developer's life, so I might as well embrace it now!

As I develop Immedialist, I will push myself to blog about the process, writing about the challenges I'm facing in order to think through problems and keep myself honest. So, stay tuned for that!
