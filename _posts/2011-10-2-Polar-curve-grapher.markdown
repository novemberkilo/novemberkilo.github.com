---
layout: post
title: A polar curve grapher
tags: [javascript, math]
---


  Polar coordinates and polar functions are remarkably beautiful to people who get mathematics.
  I think that their [graphs](http://www.wolframalpha.com/input/?i=polar+curves) are beautiful to most people. Ever since I first met them,
  I've tried to draw them on different platforms and using different languages. Wanting to know more about canvas and javascript,
  I thought that it would be interesting to implement a polar curve grapher using these.


  A quick reminder - the [polar coordinates](http://www.wolframalpha.com/input/?i=polar+coordinates) `( r, t )` of a point
  are expressed in terms of the distance `r` of the point from the origin and the angle `t`
  that the line connecting the point to the origin makes with the `x`-axis. Also, the way you get from a set of polar coordinates `(r, t)`
  to cartesian coordinates `(x, y)` is:

        x = r * cos(t), y = r * sin(t)

#### Background


  Earlier on this year I had asked [Gabe Hollombe](http://avantbard.com) for advice on things I could do to become a better programmer.
  Around the time of the conversation, I was particularly aware of my lack of knowledge of
  javascript and so, we chose my polar-curve-grapher as a project for Gabe to mentor me through.


  I started off by checking out the landscape of javascript based graphing tools and apps and came across the
  following:

1. Kevin Mehall's [EquationExplorer](http://kevinmehall.net/p/equationexplorer)
1. aanthony's [graph.tk](http://graph.tk)


  Given my relative lack of experience with javascript, I found both apps difficult to understand.
  So, I tried to figure out how to do it by myself.  I looked for javascript libraries I could use to simply plot a bunch of points
  and found a lot of stuff for charts but relatively few that would just take an array of
  points and plot them. Here are some that fit the bill:

1. [jQplot](http://www.jqplot.com/)
1. Keith Lawler's [raphael.graphing.js](https://github.com/lawler/raphael.graphing)


  I played around with jqplot and a hard-coded function `r = f(t)` to get some quick results.  Essentially, I
  created an array of points to be plotted using the following direct approach:

        points_to_be_plotted = []
        for t in (0 ... 6*PI)
        points_to_be_plotted.push [ f(t)*cos(t), f(t)*sin(t) ]

        jqPlot( point_to_be_plotted )

And this is what jQplot did for me:

<img src='/images/Polar_function_grapher-jqplot.jpg' style='border: solid; border-color: gray; border-width: 2px' />

  This gave me a quick proof of concept and, the project had a life! I now needed to figure out how to get user input, process it, and then plot it.

#### Parsing user input


  Getting user input turned out to be straightforward - use a form in the page with a submit button and respond
  to the button click by picking up the input in javascript as a string (say) `"input_fn"`.  Use `eval("input_fn(t)")` to turn this into a javascript
  function and follow the psuedocode above to generate an array of points to be plotted (see the source of the project for the detail - more on this below).


  But what if you want the user to enter a function in
  plain, pedestrian maths (instead of the syntax required by javascript) - e.g. `2sin(x)` instead of `2 * Math.sin(x)`

  Kevin's EquationExplorer did this so I set out to understand how.
  And this lead me to the world of Top-Down-Operator-Precedence (TDOP) parsers.


  I will write more about TDOP parsers in my next post but to complete the story on the polar curve grapher, these were the
  ingredients that went into the grapher that [you can find here.](http://github.com/novemberkilo/polar-curve-grapher)

1. Parsing: Kevin Mehall's `tdop_math.js` parser (that requires `tokens.js,` a lexer written by Douglas Crockford himself)
1. Graphing: Keith Lawler's graphing library that is based on `raphael.js` and called, [raphael.graphing.js](https://github.com/lawler/raphael.graphing)


And thanks to GitHub pages, you can see it in action here: <http://novemberkilo.com/polar-curve-grapher>

  At the time of posting this I still needed to apply a coat or two of polish - I'm not doing any graceful error handling (so try not to divide by zero!) and I'm
  only winding around the origin three times. You should be able to play around though and get some pretty graphs.

  Try `r = sin(3t)` to get started with a propeller :)

#### Acknowledgements


  I had a lot of fun putting this project together and owe a debt of gratitude to [Gabe Hollombe](http://avantbard.com) for encouraging me to take this on, and for answering questions
  and providing guidance along the way.  Also, thanks to Keith Lawler and Kevin Mehall for doing the heavy lifting - this project turned out to mainly involve
  stitching together their libraries.








