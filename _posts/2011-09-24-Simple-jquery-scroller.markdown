---
layout: post
title: A simple jquery scroller
tags: [jquery]
---

  Recently we implemented a variation on a scroller. We needed to flash up a set of snippets, one by one, in the same text area of a page. Here's a [demo](/examples/jquery_scroller.html) of it in action.  You can view the source of the demo but here's a quick, plain-English explanation of how it works.

#### The markup
  In the page, the snippets are list elements `li` in a `section`. You can make them definition elements or whatever you like. The section has an `id` so we can find it from javascript - I've called it `snippets` in the example.

#### The CSS
  Pretty much the only style elements that are important to the correct
operation of the scroller is `display: none` on the list elements.
This ensures that all list elements start out being invisible.  Having
`overflow: hidden` on the `section` makes it easier to deal with any
stuttering in the javascript loop shown below.

#### The javascript

We're using jQuery so make sure you've got that loaded.  And here's the
javascript:

{% highlight javascript %}

    $().ready(function(){
      var loopSnippets = function() {
        var snippets = $("li", $("#snippets"));

        snippets.each( function(index, element) {
          $(element).delay( index*5000 ).fadeIn(100).delay(4500).fadeOut(200);
          if (index == snippets.length-1) { setTimeout( loopSnippets, index*5000 + 5000 ) }
        });
      };
      loopSnippets();
    });

{% endhighlight %}

So this is where all the action is of course. The variable `snippets`
collects all the list items and then we iterate over this collection,
showing each list item and then hiding them again (using `fadeIn` and
`fadeOut`).

What you have to wrap your head around is that all of the code gets
executed in one shot, so to get items to display in sequence and to loop
around again, you have to show-and-hide each item at different times.

The trick is to use `delay` and the index of the item in the array to calculate the amount of time we need to wait before kicking off the show-and-hide routine for the item.

Finally, to loop back and start over at the top, we use `setTimeout` to
kick off the `loopSnippets` function again. Again, use the length of
`snippets` and the amount of time used up in showing and hiding each
element to figure out how long you have to wait before setting off
`loopSnippets` again.

Keep in mind that the above code works irrespective of the length of the
list of snippets - which is of course what you want if the snippets are
dynamically loaded.

#### In summary

Use `delay` and the index of elements in an array if you want to
operate on each element in sequence. Use `setTimeout` to repeat a
function call at specific time intervals.

Thanks go to **Josh Kunzmann** and **Damien Le Berrigaud** for collaborating
on this and showing me how to use `delay` and `siteTimeout` creatively!
