---
title: Case Statements and Strict Equality
layout: post
date: 2014-09-16 9:59:28
blog: true
---

I was playing around with an implementation of categories in   [Immedialist]({% post_url 2014-08-18-smarter_todo_list %}) that involved a type-checking case statement in a module. I know, I know, probably not the greatest idea; but that's not the point of this post.

## First Pass

In order to see if I could get my ``Movie#category`` spec to pass with this new implementation, I wrote the case statement like this:

```ruby
def category
  case self.class
  when Movie
    :film
  end
end
```

The method checks the class of the media type instance that is calling it (Movie, Book, Actor, etc.) and returns the correct category symbol. Pretty straightforward, right? This should totally pass.

But it doesn't!

![Failing Movie Spect]({{ site.url }}/assets/images/failing-movie-spec.png)

What I discovered upon debugging is that Ruby case statements use ``===`` to determine equality, and ``self.class === Movie`` returns false even when self is a  ``Movie`` instance. Thus, the case statement returned ``nil``.

## Classes and the === Operator

I had never really looked too closely at ``===`` until now; I had just assumed that, like in JavaScript, it was a stricter version of ``==``, and I stayed away from it. But it turns out that Ruby's use of ``===`` is different from JavaScript's in that it's more flexible and can be provided meaningful semantics depending on the class employing it.

This is what Dave Thomas has to say in *Programming Ruby 1.9* on ``===`` in ``Class``:

> Ruby classes are instances of class ``Class``. The === operator is defined in Class to test whether the argument is an instance of the receiver or one of its superclasses. So (abandoning the benefits of polymorphism and bringing the gods of refactoring down around your ears), you can test the class of objects. (131)

## The Solution

This helpful quote, along with recalling single-line case statement syntax, led to this refactoring:

```ruby
def category
  case self
  when Movie then :film
  end
end
```

This time, I'm taking advantage of the implementation of ``Class#===``, which tests whether ``self`` is an instance of ``Movie``. Cleaner code, and it passes:

![Passing Movie Spec]({{ site.url }}/assets/images/passing-movie-spec.png)

I'm still deciding whether I'm going to ignore programming best practice and move forward with the case statement or try a different approach, but at least I learned more about Ruby's case statement and ``===`` in the meantime!
