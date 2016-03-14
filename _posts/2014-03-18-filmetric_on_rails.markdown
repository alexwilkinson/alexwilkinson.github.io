---
title: Filmetric on Rails
layout: post
date: 2014-03-18 13:03:15
blog: true
---

As soon as we started learning Ruby on Rails at the Flatiron School, I knew that I needed a fun side project to start practicing all of these new patterns. Luckily for me, I had a large database of movies, actors, genres, and directors just begging to be played around with.

I had built this database for my group presentation week, practicing web scraping, API querying, Active Record associations, and join tables in order to calculate "filmetric" scores for movies (the difference between the critics and audience scores on Rotten Tomatoes). It was all well and good to have built a CLI to casually search for a movie's filmetric, but I knew that Filmetric would reach its full potential only once it was implemented as a website. Now that I had some basic rails knowledge under my belt, it was time to fulfill that dream!

## Step 1: Databases

After initializing a new rails project, I copied my database file from my old project over to the db folder of my fresh rails directory. Unlike most projects we have been working on thus far at the school, my database was very important to me; it represented a week of IMDB scraping and Rotten Tomatoes querying that I did not want to redo. Using this collection of movie data as my development database, at least in these early stages, was out of the question. So I had to quickly learn how to seamlessly transition among multiple rails environments as I worked.

As you might expect, Rails makes this pretty easy! You can prepend most rails commands with RAILS_ENV=(environment_name) in order to keep your environments distinct. I could test out schema changes and play with models on my development database before doing anything to my static data set, which I established as my production database.

{% highlight bash %}
RAILS_ENV=development rails console
RAILS_ENV=test rake db:migrate
RAILS_ENV=production rails server
{% endhighlight %}

This led to problems when I wanted to render my views using my full data set. It turns out that the asset pipeline and production don't play nicely together, and so none of my styles applied on my production server. I ended up just making my development database a copy of my production to get around this.

I also discovered a very useful option to pass to the default db:migrate rake task after running into another issue. Because I had copied my film database over from an old project, the migrations that had been applied to it were different from my newly generated Rails migrations, even though both sets result in identical schemas. This meant that when I wanted to run a new migration on my old database, I couldn't because rails would refuse to run my previous migrations.

Instead of writing a new task just to run one migration, I ran this:

{% highlight bash %}
RAILS_ENV=production rake db:migrate VERSION=20140313144836
{% endhighlight %}

That number is the prepended time stamp for the migration in question, which allows me to run just that migration on the database. Perfect!

## Step 2: Models

As I ported over my model files from my old project into Rails, I realized just how helpful it is to keep your models interface independent. On my old project, my models supported a basic CLI, while on Rails they are feeding data to a relatively complex web interface. Yet that transition couldn't make a difference in the world to my models, which made my job all the easier. Separation of concerns at its finest.

I didn't have to add any validations to my models because no one will be posting new movies, actors, genres, or directors on my site. Lucky me!

## Step 3: TTD

I wanted to take a bit more of a test-driven approach than I did on the first iteration of this project, so I did my best to at least write minimal specs for each part of my growing Rails application.

The best way to start testing your rails app is to use the [rspec-rails] (https://github.com/rspec/rspec-rails) gem. Put it in your gemfile, bundle, and then run `rails generate rspec:install` in order to initialize a spec folder.

Once installed, RSpec files will be generated instead of Test::Unit files when rails generators are executed, saving the trouble of flagging such commands with --no-test-framework. Crazy things happen when you generate scaffolds under this system; try it out yourself! (Hint: lots of free tests.)

You are also provided with RSpec generators, which stub out RSpect directories and files for you. For example,

```ruby
rails generate rspec:model User
```

will create a models director in the spec directory if it doesn't already exist, as well as a user_model_spec.rb file. You can also run generators for your other favorites, such as controller and view specs, but I couldn't get the generator to stub out feature specs for me even though the documentation claims this is possible. Oh well!

More importantly, the gem also provides some new RSpec methods to play with. Testing was very similar to Sinatra, with some minor differences to reflect the shift in platform. For example, get requests within the spec are executed on actions rather than routes.

Sinatra:

{% highlight ruby %}
describe "GET /actors" do
    before do
      get '/actors'
    end

    it "responds with a 200 status code" do
      expect(last_response).to be_ok
    end
end
{% endhighlight %}

Rails:

{% highlight ruby %}
describe "GET #index" do
    before do
      get :index
    end
    
    it "responds with a 200 status code" do
      expect(response).to be_success
    end
end
{% endhighlight %}

The rspec-rails gem also allows you to test view rendering in a controller spec, but not by default. According to the documentation, this is done “encourage more isolated testing.” Hard to argue with that, but it's easy enough to fix in case you're a rebel; just place "render_views" on the first line after a describe block. You could also just write view specs, but hey, who am I to judge!

I highly recommend skimming through all of the [documentation] (https://relishapp.com/rspec/rspec-rails/v/3-0/docs) yourself. They provide great examples of all of the different rails-rspec methods at your disposal and some good patterns to try out in your own tests.

## Step 4: Search

One aspect of this project that I was worried about was implementing search. This functionality was central to my project, so I knew it was important to get it right from the beginning. I ultimately decided to write a separate Searches controller to handle incoming search requests and query each of my models for appropriate results. That way, each of my resource controllers could remain clean, responsible solely for displaying individual resources while my Searches controller did the heavy lifting.

I knew that I should make searches GET requests to follow best practice, even though the searches in my app would be hardly robust enough to justify bookmarking. But how was I going to determine with type of resource the user was looking for? Would I be able to somehow design my app to recognize “Brad Pitt” as an actor rather than a genre or movie? And what about directors who are also actors? I could’ve displayed all results from each resource for a given query, but that sounded resource-intensive and not very user-friendly.

Since I'm not Google, I decided to make it easier on myself and provided a form select containing all four of the site's categories. That way, the user could explicitly tell me through params which resource she wanted me to look up, that she was interested in Ben Affleck as a director rather than as an actor. Which would make sense, because he _is_ a better director! Just compare the filmetrics! But I'm getting ahead of myself...

I handled fuzzy search matching (“Iron Man” => “Iron Man”, “Iron Man 2”, “Iron Man 3”) by taking advantage of Active Record and Sequel. I also built in the ability for my app to go directly to the result when there was only one match.

{% highlight ruby %}
case params[:category]
      when "Movie"
        @results = Movie.where('title LIKE ?', "%#{params[:q]}%")
        redirect_to movie_path(@results.first) if @results.count == 1
      when "Actor"
        @results = Actor.where('name LIKE ?', "%#{params[:q]}%")
        redirect_to actor_path(@results.first) if @results.count == 1
      when "Director"
        @results = Director.where('name LIKE ?', "%#{params[:q]}%")
        redirect_to director_path(@results.first) if @results.count == 1
      when "Genre"
        @results = Genre.where('name LIKE ?', "%#{params[:q]}%")
        redirect_to genre_path(@results.first) if @results.count == 1
{% endhighlight %}

Nothing terribly profound, but I was proud of it. And yes, I know that this could be totally refactored. I'll get to it soon, promise!

## Step 5: Front End

I ended up spending a little more time on making it pretty than I intended to, partially because I love obsessing over front-end implementation. Luckily, integrating bootstrap wasn't terribly difficult. I just used the boostrap-sass gem, added a few lines to some files in my assets directory, and I was off to the races!

I would explain what my procedure was, but I have subsequently learned of an easier way: read the [documentation](https://github.com/twbs/bootstrap-sass)! The github page explains the process better than the random internet guides, promise. The key section is [Usage](https://github.com/twbs/bootstrap-sass#usage), which explains where to import the css and javascript from bootstrap in your asset pipeline.

The best way to get started using bootstrap quickly without a theme is to take advantage of bootstrap grid layouts. Having wrestled with stubborn divs for longer than I'd like to admit, floating helplessly left and right and positioning absolutely and relatively like a madman, it feels pretty great to tuck page elements neatly into column-spanning modules and rows. It's an awesome way to quickly bring some order and consistency to your layout without too much effort.

![Bootstrap Alerts]({{ site.url }}/assets/bootstrap-alerts.png)

Also, the bootstrap-styled rails flash messages are really pretty, and you can make them dismissible, too! Pro Tip: Positioning them absolutely is a great way to have them hover on top of the page instead of shoving everything beneath it down.

## Step 6: Chartkick

The final piece of the puzzle was figuring out how to cleanly and effectively insert charts into my views. I wanted to display the filmetrics of all of a director or actor's movies as a line chart in order to better contextualize each of their average filmetrics.

{% highlight ruby %}
<% @data = [].tap do |data| %>
        <% @actor.movies.map do |movie| %>
          <% data << [Time.new(movie.release_year), movie.filmetric] %>
        <% end %>
      <% end %>

     <%= line_chart @data, library: { tooltip: { trigger: 'none' },
     title: "Movies by Filmetric" } %>
{% endhighlight %}

It turned out that [Chartkick] (http://chartkick.com/) was one of the easiest ways to do so without resorting to front-end skills way beyond my abilities. I found it easiest to insert the data into the chart using a pre-generated hash of values, or x,y coordinates, for the graph. The x-coordinates needed to be date objects, so I used `Time.new` to format the release_years in each movie.

![Filmetric Graph]({{ site.url }}/assets/filmetric-graph.png)

I had two major issues with this implementation, unfortunately. One, I couldn't figure out how to get the “tooltips” on each data point to display a movie title, which meant that I disabled them entirely. Two, I only stored movie release dates as years, meaning that when actors or directors inevitably had movies in the same year, the data points overlapped, making the graphs less robust. Re-querying the data will rectify the second issue, and the first can hopefully be resolved by just understanding how to use Chartkick better.

## Conclusion

I'm super excited by how far my paltry rails knowledge has gotten me so far. It is honestly surreal to surf my Filmetric app and reflect on the fact that I built it myself, a task that just a few months ago I always felt would be someone else's job. It's no where near finished, but I can't wait to keep returning to the project with new knowledge and new ideas over the duration of the Flatiron School.






