---
title: Setting Secure ENV Variables on Travis
layout: post
date: 2014-05-02  0:43:50
blog: true
---

I’ve been busy packaging one of my earliest Rails projects, [Filmetric](https://github.com/alexwilkinson/filmetric_rails) -- writing more tests, removing unnecessary files, and refactoring. I discovered a lot of hidden bugs in the app along the way, and reaffirmed my belief that tests are critical, even for small projects. They’re worth writing if only for the peace of mind that comes with being able to run your test suite after ever feature change instead of clicking randomly around your site.

Obvious, I know. But I feel like I relearn this lesson every time “just a quick little feature update” turns into a bug-hunting nightmare because I didn’t bother to write controller or model tests for edge cases.

I also hooked up Filmetric with [Travis]("https://travis-ci.org") so that I could start getting used to using a continuous integration service. It took a few tries to get my build to pass. After figuring out how to properly set up my [YML file](http://docs.travis-ci.com/user/getting-started/) and get Postgres [working](http://docs.travis-ci.com/user/using-postgresql/#Using-PostgreSQL-in-your-Builds), my tests were still failing because Travis didn’t know about my [Figaro](https://github.com/laserlemon/figaro) application.yml file.

This was important because several critical tests depend on my Rotten Tomatoes API key. It took me a little longer than I would’ve liked to figure out the solution, so I thought I would provide the steps you need to take to get environment variables up on Travis.

Download the [Travis CLI](https://github.com/travis-ci/travis.rb):

{% highlight bash %}
  gem install travis -v 1.6.10 --no-rdoc --no-ri
{% endhighlight %}

Check your Travis version after this step just in case. In my case, it prompted me to install bash completion.

{% highlight bash %}
  travis version
{% endhighlight %}

Use the [encrypt command](https://github.com/travis-ci/travis.rb#encrypt) to secure your environment variable(s). You can automatically add them to your travis.yml with the --add option (recommended: why not save a step?).

{% highlight bash %}
  travis encrypt FOO=bar --add
{% endhighlight %}

If you have multiple variables in one file, you could also try using this handy trick: piping the file into the command and then using the --split option.

{% highlight bash %}
  cat .env | travis encrypt --split
{% endhighlight %}

That’s it! All you need to do is push with your travis.yml changes, and you’re all set.

Now, back to increasing my test coverage!
