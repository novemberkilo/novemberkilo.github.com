---
layout: post
title: A Top Down Operator Precedence parser for algebraic expressions
tags: [javascript, ruby, math]
---

In my [previous post](/2011/10/Polar-curve-grapher/) I wrote about a polar curve grapher and its use of Kevin Mehall's `tdop_math.js` to parse a user input algebraic expression into a
javascript eval(uable) string.  I ended up spending some time with `tdop_math.js` - you can find the source [here](https://github.com/kevinmehall/EquationExplorer/blob/master/tdop_math.js) -
and I fell into the magical rabbit hole of Top Down Operator Precedence (TDOP) parsers. This post describes how I ended up writing a
TDOP parser for algebraic expressions in Ruby.

Some facts:
- Vaughan Pratt first wrote about Top Down Operator Precedence parsers in 1973
- Douglas Crockford wrote about them in Beautiful Code, and in [an article](http://javascript.crockford.com/tdop/tdop.html) that includes an implementation of a parser for a Simplified Javascript - he went on to
use this in [JSlint](http://jslint.com)

#### Overview of TDOP parsers

In a nutshell, Top Down Operator Precedence parsers are similar to [Recursive Descent parsers,](http://en.wikipedia.org/wiki/Recursive_descent_parser) but, as claimed by Vaughan Pratt,
simpler to understand, implement, and providing of better performance. Borrowing from [this](http://eli.thegreenplace.net/2010/01/02/top-down-operator-precedence-parsing/)
excellent article by Eli Bendersky, the main features of a TDOP parser are:

- A _binding power_ mechanism to handle precedence levels - e.g. the binding power of multiplication `'*'` is higher than that of addition `'+'` in arithmetic
- A means of implementing different functionality of tokens depending on their position relative to their neighbors: _prefix_ or _infix._ For example, the
minus `'-'` operator is behaving as a prefix operator in `-2` but in its infix form, it is subtraction `4 - 2.`
- As opposed to classic Recursive Descent, where semantic actions are associated with grammar rules (BNF), TDOP associates them with tokens. Tokens have
a binding power, and know how to handle themselves in infix and/or in prefix situations. Expressions are then evaluated recursively by evaluating them at each
'precedence' level and returning the result to its caller.

Compilers are difficult to understand at the best of times and most articles I have come across on TDOP parsers are by very bright people finishing (or pausing) on their journey to
get to know Pratt's work. At my current stage of comfort with TDOP parsers, I cannot come close to explaining the parser any better than
Eli Bendersky.

Eli's [article,](http://eli.thegreenplace.net/2010/01/02/top-down-operator-precedence-parsing/) elaborates on the features described above
and explains the parser by setting one up to handle simple arithmetic. He then works through a simple example that illustrates it in
action as it parses the expression `3 + 1 * 2 * 4 + 5.`

For further detail on TDOP parsers, I refer the interested reader to this and Douglas Crockford's article referenced above.

### TDOP and Ruby

As I was working on the [polar curve grapher](2011/10/Polar-curve-grapher/) I got curious about implementing `tdop_math.js` in Ruby. I studied the
source of `tdop_math.js,` and its tokeniser `tokens.js,` and felt that I had a decent understanding of its structure and operation (improving my Javascript skills
along the way!). However, a like-for-like implementation in Ruby seemed out of the question. Well, it seemed like a hard proposition that would involve
first developing new metaprogramming skills in Ruby!

And then I ran into Glen Vandenburg's project `radish,` which is now called [`smithereen`.](http://github.com/glv/smithereen)

#### Smithereen

Smithereen is a result of Glen Vandenburg's meditation on TDOP parsers, during which he decided to write an implementation in Ruby.
He has implemented a _library for building Top Down Operator Precedence parsers,_ using Ruby idioms that were completely unfamiliar to me, but
that provided an excellent opportunity to learn. As Glen says, he has _tried to provide a well-factored set of tools to help with
the core TDOP algorithm and several related problems,_ and so I decided to fork Smithereen with the intention of helping with its development, and,
using it to implement `tdop_math` in Ruby.

Smithereen comes with two samples, an arithmetic parser, and the Simplified Javascript parser.  I found it quite difficult to get started with Smithereen
and asked Glen a question or two. To my delight, he helped me get started and I found this to be so inspiring that I made rapid progress towards
implementing a simple scientific calculator.  This is available on [my fork of Smithereen](http://github.com/novemberkilo/smithereen/tree/develop)
as [`equation_parser.rb`](https://github.com/novemberkilo/smithereen/blob/develop/samples/equation_parser.rb)

`equation_parser.rb` will, for instance, take strings such as `sin(cos(2pi))` and return `0.84147`

By way of explanation, it has hardcoded hashes that declare functions and constants, and these are tokenised as `names` and given a high
binding power.

{% highlight ruby %}

    fns = {
      'sin'  => Proc.new { |x| Math.sin(x) },
      'cos'  => Proc.new { |x| Math.cos(x) },
      'tan'  => Proc.new { |x| Math.tan(x) },
      'sqrt' => Proc.new { |x| Math.sqrt(x) },
      'exp'  => Proc.new { |x| Math.exp(x) }
      }

    CONSTANTS = {
      'pi' => Math::PI,
      'e'  => Math::E
      }

{% endhighlight %}

The `infix` method on the token `(` checks for the presence of a function or a constant (essentially a key in the above hashes),
and looks up the corresponding value for the definition of the function. Smithereen takes care of the rest.  Well, that is, after tokens for
the usual arithmetic operators are defined, with infix and prefix methods as described above.

#### tdop_math.rb

Completing my goal to create a Ruby equivalent to `tdop_math.js,` I've taken the work in the example `equation_parser.rb`
and used it to create a Ruby gem called `tdop_math.` This takes an algebraic expression, parses it, and returns a string of Ruby that represents
the expression.

Because the `tdop_math` gem depends on `smithereen` (which has yet to be released), it is not available as a built gem.

As far as I know, if you want to install this, you are going to need to pull down my [repo](http://github.com/novemberkilo/tdop_math) and
`include '<path-to-tdop_math.rb>'` manually. Once you have, you use it as follows:

{% highlight ruby %}
    u = "sin(cos(2x))"  ## or some user input algebraic expression
    t = TDOPMath::Parser.new(u)
    f = t.parse   ## f == "Math.sin(Math.cos(2 * x))"
{% endhighlight %}

`tdop_math` reserves symbols `x`,`y` and `t` as variables.  It also
provides a basic list of functions, namely, `sin`, `cos`, `tan`, `sqrt`
and `exp`.  If you need to provide an alternate list of variables and
functions, you can do so as follows:

{% highlight ruby %}
    u = "baz(foo(2a))"  ## or some user input algrebraic expression
    t = TDOPMath::Parser.new(u,
                             :vars => ['a'],
                             :functions => {
                                'foo' => 'Foo::foo',
                                'baz' => 'Baz::baz'
                              })
    f = t.parse   ## f == "Baz::baz(Foo::foo(2 * a))"
{% endhighlight %}

You can use `tdop_math` wherever you need a user to input an algebraic expression, comprising of a known set of functions and variables. This
could find use in apps in maths education or science/data analytics.  I hope the community finds it useful, although my motivation was mainly
to build it as an exercise, along my path to becoming a better programmer.  I've made a gem!

