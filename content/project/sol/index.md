---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Klondike Solitaire"
summary: "A solitaire card game for the Commodore 64."
authors:
  - andrew
tags:
  - retrocomputing
  - assembly
  - 6502
  - c64
categories: []
date: 2021-02-11T06:15:41+09:00

# Optional external URL for project (replaces project detail page).
external_link: ""

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_code: ""
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---
Klondike Solitaire is an implementation of the
[famous solitaire card game](https://en.wikipedia.org/wiki/Klondike_(solitaire))
for the Commodore 64 written in 6502 assembly.
It currently only supports text mode, but I plan to add support for better
graphics in a future release.
I am also considering porting it to the new Commander X-16 system.

I wrote Klondike Solitaire primarily as an intermediate introduction to 6502
assembler, but it is quite playable and enjoyable.

![Welcome Screen](welcome.png)

You can choose to play in easy
mode (where one card is drawn from the deck each time) or hard mode
(where three cards are drawn each time).

Cards are moved by first selecting a source (`D` for the deck or a number
from `1` - `7` for a stack) and then a destination (`P` for the piles or
`1` - `7` for the stacks.)
Stacks of cards can be moved all at once when one of the cards
in the stack matches the destination card.
Pressing `C` will draw another card from the deck.

If you would like to try it out, the current version can be downloaded here:

<p style="text-align: center"><a href="sol.prg">{{< icon name="download" pack="fas" >}} sol.prg</a></p>

If you enjoy the game, please consider [buying me a coffee](https://ko-fi.com/andrewyoung).
