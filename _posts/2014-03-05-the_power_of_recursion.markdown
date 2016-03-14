---
title: The Power of Recursion
layout: post
date: 2014-03-05 15:11:46
blog: true
---

If you had told me a week ago that I would even be attempting to write a blog post on recursion, I probably would've laughed in your face. And then I would've panicked about how I could possibly write about recursion, which wouldn't have been funny.

Up until this point, the fifth week of the Flatiron School Ruby program, I lived in fear that I would one day have to write a recursive program in order to become a hirable programmer. Writing programs has been hard enough; I couldn't even imagine building one that called itself! What does that even mean? And why would it ever be useful for me to know?

I expressed my reservations to a fellow Flatiron student, [Daniel Kronovet](http://kr0nos4piens.wordpress.com/), and he very kindly sat down with me and explained the basics. This post will be an attempt at recounting what I learned from him, as well as tacking on what I've discovered about recursions since. Hopefully it will be helpful to those of you who are scared of these cryptic functions!

(I still am, by the way, lest you think I have deigned myself some sort of expert just because I know how to discover fibonacci numbers. More on that in a sec.)

## Function(function(function(function)))

So what are recursions, anyway? Here's my (probably not fully accurate) understanding of a common use case for them. Not exactly a defintion, I know, but you can go to Wikipedia for that!

A recursive function is one that implements an algorithim to solve a problem that, in the process, requires calling itself. I like to think of recursive functions as a proxy for the programmer herself. Rather than the coder implementing each step in the process of solving a problem, she instead identifies a higher-level algorithm that will lead to a solution and tells the computer to run it. This requires a certain level of faith, particularly for complex recursive functions. You don't necessarily need to know what the function does at every step (unless it isn't working...) as long as you have written accurate instructions for the computer to follow.

To write a recursive solution to a problem, you need to identify and implement three things:

 - The simplest, or base, case(s) for a solution.
 - How to break down a problem into its simplest components.
 - How to return the solution you are looking for by synthesizing the results of your recursions.

## Fibonacci to the Rescue

That wasn't so helpful, I know. Luckily for you, I have an example ready!

{% highlight ruby %}
def fib n
  return n if n <= 0
  n <= 2 ? 1 : (fib(n-1)) + (fib(n-2))
end
{% endhighlight %}

As you could probably guess, this function finds the fibonacci number at the nth point in the fibonacci sequence. For those who need a quick refresher (I certainly did!), fibonacci numbers are computed from a sequence where each subsequent number is the sum of the previous two.

{% highlight ruby %}
1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144...
{% endhighlight %}

It's easy to find the first few fibonacci numbers, but it rapidly gets quite difficult. Sounds like something a computer can solve!

Let's analyze this method based on the criteria for recursive functions described above. In order to do so, I'm going to write it out on a few more lines so it's easier to see, even though it's a bit less svelte. (Yes, that's the techinical term.)

{% highlight ruby lineos %}
def fib n
  if n <= 0
    n
  elsif n <= 2
    1
  else
    (fib(n-1)) + (fib(n-2))
  end
end
{% endhighlight %}

The first two branches of the control flow represent the base cases for solving this problem: n is <= 0, in which case we return n, or n is 1 or 2, in which case we return 1. On line 7, we break down the problem, expressing what it really means to calculate a fibonacci number at nth position in the sequence. Since we know that any fibonacci number is calculated by the sum of the two previous numbers in the sequence, it follows that, to find the the fibonacci number corresponding to the nth position, we have to find the fibonacci numbers at the two previous positions and add them together.

At this level of abstraction, we can express this idea recursively. At fib(n-1) and at fib(n-2) we run our function again, determining whether (n-1) or (n-2) is <= 2, or if we need to execute our recursive calls once again, discovering each previous fibonacci number in turn until we arrive at our base cases. The summing operation also conveniently deals with our third condition: figuring out a way to return the sythesis of all of our recursion calls.

This was really hard to wrap my head around at first, but that's kind of the point of recursion. Once you understand how to solve for any fibonacci number, you can solve for the rest without even fully understanding what's going on at each step. In this case though, there are very handy images that cleanly map out what this function accomplishes.

![Fibonacci Tree]({{ site.url }}/assets/fib.png)

So for each phrase on line 7, this function recursively determines all of the fibonacci numbers that form it until reaching fib(2) or fib(1), which it already knows equals 1. All of those recursive calls are summed, revealing our answer. All this function does is add the right number of 1's together!

## All Together Now

As long as you can identify the base conditions to solve for and the basic algorithm necessary for solving a problem, you can probably write a recursive function. I say "probably" because there is an important caveat to this whole recursion thing. There is a computer science adage that all problems that can be solved recursively can be solved linearly, but not all problems that can be solved linearly can be solved recursively.(Important sub-caveat: I don't have a CS degree! So that may not be exactly what the adage/law/principle is.)

Basically, this means that you never have to solve any problem recursively if you're not comfortable doing so. But if you're stuck on a problem, try applying the above three principles and see what happens! Also, recursive solutions can be super elegant, and it feels pretty awesome when you come up with one. So why not try it out!

## Bonus: Solving a Maze with Recursion

I really wanted to practice recursion after finally beginning to understand it, so I jumped at the opportunity to use it to solve the maze problem. Yesterday, my team and I did just that, and I have included the result below. You can check out the full set of files [here](https://github.com/flatiron-school-students/maze-solver-ruby-004/pull/7/files).

{% highlight ruby linenos %}
def search(current, history)
 possibilities = next_nodes(current, history)
 if possibilities.empty?
   nil
 elsif possibilities.values.include?("@")
   [current, possibilities.key("@")]
 else
   possible_paths = possibilities.keys.map do |move|
    search(move, history << current)
   end
   valid_paths = possible_arrays.compact.select{|path| valid_path?(path)}
   shortest_path = valid_paths.min {|a, b| a.length <=> b.length}
   if shortest_path.nil?
     nil
   else
     shortest_path.unshift(current)
   end
 end
end

def next_nodes(current, history)
 hash = {
  [current[0]-1, current[1]] => maze_array[current[0]-1][current[1]],
  [current[0], current[1]+1] => maze_array[current[0]][current[1]+1],
  [current[0]+1, current[1]] => maze_array[current[0]+1][current[1]],
  [current[0], current[1]-1] => maze_array[current[0]][current[1]-1]
  }.reject {|k, v| history.include?(k) || v == "#"}
end

def valid_path?(path_array)
 path_array[-1] == [7, 10]
end
{% endhighlight %}

To be honest, I still have some trouble explaining this solution. For one thing, getting a computer to solve a maze for you is tough, and if I had to solve it again right now in a completely different way, it would take me a long time. This was also very much a team effort; without them, I don't think I would've been able to do it on my own. But hey, this is exactly why I'm writing a blog post: to get better. So let's give it a shot!

## Dungeon Crawling

After setting up the solution by transforming a maze in string form to an array from which coordinates can be extrapolated, we began the process by calling the search method on the coordinate of the maze's starting point.

Line 2 is where a lot of the action happens. We pass the coordinates we are currently on as one argument and an array of previous nodes visited as the second and use the next_nodes method to figure out which adjacent spaces the computer could move into. The next_nodes method examines what is contained in the spots above, left, right, and below the current spot and returns an array of options, filtering out walls of the maze and spots visited previously.

Once we get our possibilities returned, we deal with the two base conditions of our problem on lines 3-6: finding a dead end in the maze and reaching the end of the maze. If neither of those cases are true, we reach the algorithmic aspect of the method on line 8, where we present the way to solve for any space of the maze. This involves returning an array of all of the coordinates based on examining each remaining space of the maze recursively, checking them one at a time for whether there is a dead end, an exit, or still more possibilities.

The key to understanding what's happening here is to recall the fib method, where all the subsequent recursions can be thought to occur within the parentheses that enclose fib(n-1) and fib(n-2). At this point in our search method, the program yields to the block for a while, with each new call to search waiting for the next recursion to complete until the end of the maze is discovered. The first run through the search method *does not continue* until the block's recursions are finished.

When the "@" is included within an adjacent space, the block returns an array of the current position and the coordinate of "@." At this point the recursions cease, and possible_paths contains all of the paths the computer tried to reach the goal.

Quick tangent: this is actually a depth-based search rather than a breath-based search, which is not what we intended on doing upon starting the problem. This makes sense though, because by its very nature, a recursive function operates like a laser, honing in on a particular direction until it finds what its looking for rather than casting the wide net of a breath-based search. Even more interestingly, because we looked to top and right adjacent nodes first in the next_nodes method, we likely directed our MazeSolver more quickly toward the solution. In case you were wondering, I tested out switching the position checks around, and MazeSolver still returns the correct solution regardless of the order. I also did a rough benchmark and couldn't get a conclusive answer as to whether starting with the bottom or the top was faster. Perhaps with a bigger maze it would be clearer!

Moving on, we gather viable paths to the "@" by filtering out nils and any paths that do not contain the exit coordinates. We needed to take both of these actions even though the nil values signified a dead end because passing nil into future methods got us into some trouble. Finally, we find the shortest path through a simple .min comparison, place in one last check for nil, and then return an array of the shortest path with the node we ended on unshifted to the front.

## By the Way, Recursion Is Really Cool

And that's it! Even for a problem as complicated as solving a maze, it's amazing how far you can get just by breaking down a problem to its simplest components and letting the computer do the rest of the work. I'm looking forward to seeing what future problems can be solved using recursion, and I hope this post at least got you interested in trying it out yourself!
