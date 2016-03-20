---
title: Working with Google Books
layout: post
date: 2014-04-02  8:46:58
blog: true
---

I've been working on a new site that serves up books that you can read, and then let's you "like" or "dislike" them to get new options. My group is hoping to eventually emulate the experience of browsing through a bookstore, but before any of that happens we need to wrangle the Google Books JavaScript viewer into submission.

Here are some interesting things I've learned about the [Embedded Viewer API](https://developers.google.com/books/docs/viewer/developers_guide?hl=ja) for the viewer thus far that could hopefully help someone else wishing to integrate it into a project or website.

## Displaying the Viewer

The API is basically an iFrame that can be inserted into any div. The JavaScript boilerplate code Google provides will assume that div has a specific ID (though this can obviously be changed), hooks into that div, and displays the viewer inside of it. The viewer is flexible, so the div's height and width can be customized. You can even insert it into a modal like we have, which allows for some cool effects!

The documentation assumes that you will be embedding the viewer straight into your HTML, like so:

{% highlight html %}
<!DOCTYPE html "-//W3C//DTD XHTML 1.0 Strict//EN"
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8"/>
    <title>Google Books Embedded Viewer API Example</title>
    <script type="text/javascript" src="https://www.google.com/jsapi">
    </script>
    <script type="text/javascript">
      google.load("books", "0");

      function initialize() {
        var viewer = new google.books.DefaultViewer
        (document.getElementById('viewerCanvas'));
        viewer.load('ISBN:0738531367');
      }

      google.setOnLoadCallback(initialize);
    </script>
  </head>
  <body>
    <div id="viewerCanvas" style="width: 600px; height: 500px"></div>
  </body>
</html>
{% endhighlight %}

We pretty much just followed Google by the letter to get it working in our Rails project, breaking up the code into different files where possible.

Here is the first file:

{% highlight html %}
<script type="text/javascript">
  google.load("books", "0");
    $(document).ready(function initialize(){
      $("button#search").on("click", function(e){
      $.getScript("<%= preview_path %>", function(e){
          $("#myModal").modal('toggle', function(){
          });
      });
      $("button#next").on("click", function(e){
          $.getScript("<%= preview_path %>", function(e){
            });
        });
      });
    });
</script>
{% endhighlight %}

And the second:

{% highlight javascript %}
var viewer = new google.books.DefaultViewer(document.getElementById('viewerCanvas'));
var number = <%= @isbn %>;

console.log("ISBN is"+number);
viewer.load("ISBN:<%= @isbn %>", alertNotFound);

function alertNotFound() {
    console.log("oops!");
  $.getScript("<%= preview_path %>", function(e){
  });
}
{% endhighlight %}

We couldn't get the JavaScript within the HTML script tags to load properly outside of HTML, so we decided to just stick it into a snipper that we render on our home page. We will hopefully be able to fix that soon. The libraries that the code depends on our loaded with the asset pipeline, so DHH would like that aspect of it at least!

## Fetching Books with the Viewer

However, the API is not that smart. You can't give it a book title and expect it to resolve that search properly. It will only accept numeric values, ISBNs to be more precise. So you not only need to know what ISBN you need to provide, but also whether that ISBN has a Google Books preview associated with it. The best way we discovered to accomplish that was to use the API in tandem with the Google Books API, which you can use to search by title.

We ended up settling on a [Ruby wrapper](https://github.com/zeantsoi/GoogleBooks) for the Google Books API (there was really only one that is still being actively supported anymore) and are plugging titles into it in order to grab ISBN numbers that we then plug into the Embedded Viewer API. That process works pretty well for one primary reason: the Google Books API gem is pretty smart. Because it sorts titles by relevance, if a preview exists for a certain title, the ISBN with the preview will be served up as the first search result every time. This means that we don't really have to worry about which ISBN might have a preview, because our gem takes care of that for us.

## Handling Errors

But what about when we pull up a book that definitely doesn't have a preview? It happens fairly frequently, and when we're just feeding in random titles, it's pretty inevitable. Not only that, but sometimes previews fail to load even if they do exist for the ISBN in question. The Embedded Viewer API does provide one particularly handy feature to help out with these cases. You can pass in an error function as a second argument that handles what the viewer does if it fails to render the preview. In our case, we just execute another Ajax request for
a new book, which works out pretty well.

While this setup should take care of all situations in theory, like most things in programming, that doesn't really happen in practice. We are currently struggling with one particularly pernicious edge case, in which an internal server error fires at seemingly random times while loading books. It doesn't seem to be related to particular books, which is what we thought at first. It sometimes lasts for just a couple of requests, sometimes for 30. No perceivable pattern there either.

Our number one priority is squashing this bug, so I'll be sure to update this post when we figure out what's going wrong. Hopefully it's just related to our implementation of the reader, which means these steps will work perfectly for you. But somehow I don't think that is terribly likely.
