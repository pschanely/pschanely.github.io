---
layout: post
title:  "Simple length estimation for variable-width fonts"

---

This post is much more mundane than
[CrossHair](https://github.com/pschanely/CrossHair)!
But handy, I think.

It's hard to compute the width of text in a font
because the character "i" is likely much more narrow
than the character "m".

If you just want an estimate, you can count characters.

But can we get a better estimate without needing to know the
font details?

Recently, I wanted to attempt per-character estimates, and wrote
[a quick script](https://gist.github.com/pschanely/7df86fdc4150141487d1615a0398b562)
. Here they are!:

```
{" ": 0.328, "!": 0.381, "\"": 0.475, "#": 0.655, "$": 0.655, "%": 1.069,
"&": 0.9, "'": 0.347, "(": 0.414, ")": 0.414, "*": 0.555, "+": 0.713,
",": 0.328, "-": 0.414, ".": 0.328, "/": 0.346, "0": 0.655, "1": 0.655,
"2": 0.655, "3": 0.655, "4": 0.655, "5": 0.655, "6": 0.655, "7": 0.655,
"8": 0.655, "9": 0.655, ":": 0.346, ";": 0.346, "<": 0.713, "=": 0.713,
">": 0.713, "?": 0.619, "@": 1.201, "A": 0.864, "B": 0.829, "C": 0.862,
"D": 0.897, "E": 0.793, "F": 0.724, "G": 0.931, "H": 0.897, "I": 0.381,
"J": 0.55, "K": 0.864, "L": 0.726, "M": 1.071, "N": 0.897, "O": 0.931,
"P": 0.758, "Q": 0.931, "R": 0.862, "S": 0.758, "T": 0.759, "U": 0.897,
"V": 0.864, "W": 1.173, "X": 0.864, "Y": 0.864, "Z": 0.759, "[": 0.381,
"\\": 0.346, "]": 0.381, "^": 0.583, "_": 0.655, "`": 0.347, "a": 0.619,
"b": 0.655, "c": 0.585, "d": 0.655, "e": 0.619, "f": 0.381, "g": 0.655,
"h": 0.655, "i": 0.312, "j": 0.312, "k": 0.621, "l": 0.312, "m": 1.0,
"n": 0.655, "o": 0.655, "p": 0.655, "q": 0.655, "r": 0.414, "s": 0.55,
"t": 0.346, "u": 0.655, "v": 0.621, "w": 0.897, "x": 0.621, "y": 0.621,
"z": 0.585, "{": 0.509, "|": 0.285, "}": 0.509, "~": 0.698}
```

Each value in this JSON map is a width, relative to the width of a
lowercase "m" character.
(e.g. the width of the "i" character is 0.312 times the
width of the "m" character)

This data is limited to printable ASCII characters. (the
.afm files I found don't have information for unicode, sadly)
But if you find it useful, feel free to use it in your project.



The Details
-----------

I easily found some metrics for a few fonts: "Helvetica", and "Times New Roman".
I computed a relative-width map for each, and averaged them to produce the
above map. Hopfully combining a serif and sans-sarif font gives us some diversity.

This approach ignores a lot of details that matter, like letter spacing, word
spacing, and ligatures.


How much better is it?
----------------------

Digging up metrics for a third font, "Arial", I computed the average per-character
error compared to that font.

The estimate from the map above will be **6% different** than the
actual width, on average.

On the other hand, if you use a constant character width, your estimate will be
**17% different** than the actual width.

Using the map is clearly an improvment, but it's also no substitute for working
with the real font you're using.
