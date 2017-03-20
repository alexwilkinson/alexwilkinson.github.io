---
title: Mobile First Data Schema
layout: post
date: 2017-03-20  0:09:58
blog: true
---

As primarily a server-side developer, it has been interesting adjusting to mobile development at SeeClickFix. I have learned to anticipate and gracefully recover from errors instead of throwing them as I would in a Rails app. I center my UI development around Apple's, tools, frameworks, and API's instead
of whichever JavaScript and templating libraries I wish to use. I pay more attention to memory than I ever have before.

But there is one major difference between web-first and mobile-first development that I did not expect: my approach to data storage and schemas.

## The Scenario

I am working on a companion mobile app, built in React Native, for a board game
I am developing. It is a sci-fi choose-your-own-adventure co-op game, so the app will be displaying the players' adventures as they travel through space, allowing them to choose among an array of options.

I knew that I would need an elegant, cross-platform way to manage local data storage on the device, so I opted for [Realm](https://realm.io/products/realm-mobile-database/). Since I am building the mobile app before I have a Rails API, I am hard-coding the story content to test things out ahead of time. Players start a game from the app, so I use Realm as the database system for storing new game instances and associated progress data. Even in my early version of the app, I actually create new game objects and stash them on the device, which is close to how things will really work.

In the back of my head, I figured that game data would end up being stored on the server, and the device  would just hang onto that data temporarily. But the more I built out the app, the more I realized that I shouldn't necessarily make that architecture choice. The iOS app we are building at SeeClickFix is almost entirely API-driven, so it didn't occur to me initially that there was another way to establish a relationship between mobile app and server.

## Tale of Two Databases

Rather than have my React Native app use local storage only as a temporary caching mechanism, and consider an API as the only source of truth for all data, the Realm database in my app could store the players' game data while the Rails app could store all of the story content.

It turns out that the server doesn't ever need to know about the players' progress through a game, the game's state, or what adventures they completed. All it needs to do is store the adventures and allow the app to request them (as well as provide a light CMS so that we can easily enter new stories into the database, but that's a separate topic).

The only purpose for storing player game data on the server would be for backup purposes, in case the app's local storage is wiped or if players want to load a previously-started game on a different device by logging in. But there are far easier solutions for this than adding authentication, new API endpoints, and all of the models to the Rails app, not to mention the extra client-side code for requesting and parsing the game data from the API. Instead, I could use iCloud sync on iPhones and provide other syncing options for Android, or I could even try out [Realm Mobile Platform](https://realm.io/products/realm-mobile-platform/).

But more to the point, I may never even need this feature. It's possible that I could cover the vast majority of use cases by just storing game data on the one originating device. Games should last no more than a couple of hours, and it's unlikely for a device to lose the app's local data while players would still wish to play the game they lost. I don't anticipate gaps between play sessions for a single game being that long.

## The Lesson

If I had started the Rails app before the React Native app, I may have written all of the server-side code for storing game data before seeing if that was even necessary to do for a minimum viable product. By starting with the mobile experience first, I discovered that not all data needs to be stored on the server; in fact, some data is better off being stored on the device.

Even if I end up deciding to back up and sync game data using the Rails app, I have successfully deferred that decision to a time when I have much more information and am doing it for concrete reasons. Going into this project with a mobile developer's mindset not only taught me something new, but saved me valuable time for a side project, where every minute is especially precious.
