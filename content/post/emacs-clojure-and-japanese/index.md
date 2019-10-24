---
date: "2009-11-28T13:26:04"
lastmod: "2009-11-28T13:26:04"
title: "Emacs, Clojure, and Japanese"
# Featured image
# To use, place an image named `featured.jpg/png` in your page's folder.
# Placement options: 1 = Full column width, 2 = Out-set, 3 = Screen-width
# Focal point options: Smart, Center, TopLeft, Top, TopRight, Left, Right,
#                      BottomLeft, Bottom, BottomRight
# Set `preview_only` to `true` to just use the image for thumbnails.
image:
  placement: 1
  caption: ""
  focal_point: "Center"
  preview_only: false
---
This might be proof that I'm crazy.

I'm working on a [project](http://github.com/vaelen/jpdv) for my NLP class that involves generating a [semantic vector space](http://en.wikipedia.org/wiki/Vector_space_model) for Japanese text, and I decided that this might be a good time to learn one of the LISP dialects.
I've been looking at [Clojure](http://clojure.org/) for a while now, but I hadn't taken the time to learn it before.
I must say, I'm quite impressed so far.
The fact that reading a Japanese XML document into a data structure "just works" without any tweaking is pretty nice.
I'm still coming to grips with functional programming, but I'm liking it so far.
And the best thing as far as I'm concerned is that Clojure code can easily be made [massively parallel](http://clojure.org/concurrent_programming) thanks to all of its data types being immutable (ala Erlang).
That will help me out a lot when I need to run my code on multi-core / multi-processor machines (which I often have to do because of the sheer amount of data being crunched.)
