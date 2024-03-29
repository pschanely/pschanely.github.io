---
layout: post
title:  "Handling Nondeterminism in CrossHair"

---

CrossHair v0.0.23 was released today with an important new capability for
dealing with nondeterministic behavior.
Let's get into it.

## Handling Nondeterminism

CrossHair will fail or even yield a incorrect result if you run it on
nondeterministic code.

Here are some common things that cause nondeterminism:
 * Code that uses the current time, e.g. `time.time()`.
 * Code that uses random numbers, e.g. `random.randint(...)`
 * Code that uses a cache, e.g. `functools.lru_cache(...)`
 * Code that reads data from the disk or network.

As you might imagine, it's very common to use functions like these.
Now, with the work that [Loïc Montandon](https://github.com/lmontand) just integrated,
we have ways to check properties even when nondeterministic behavior exists.

By identifing the core functions that contain the nondeterminism, we can
now tell CrossHair to skip them and instead allow the function to return any value.

One can also apply contracts to these functions to constrain the parameters and
return value.

What's more, this capability is part of the plugin system, so you can create this kind of
behavior for 3rd party modules in a reusable & shareable fashion.

You can read more about this capability in the updated
[plugins docs](https://crosshair.readthedocs.io/en/latest/plugins.html#adding-contracts-to-external-functions)!


## What's Up Next?

* I'm going to do a pass through the standard library and patch the most common sources
  of nondeterminism (`time.time`, `random`, etc) using Loïc's work.
  You can follow that effort
  [in this issue](https://github.com/pschanely/CrossHair/issues/162).
* Separately, I'm making some progress refactoring hypothesis to
  better support the CrossHair integration.
  Importantly, this will give us access to a large number of
  properties to test CrossHair.
  And perhaps start a bug trophy list!
  You can follow this effort
  [here](https://github.com/HypothesisWorks/hypothesis/issues/3086).

As always, thanks for taking some time to share this journey with me. - Phil
