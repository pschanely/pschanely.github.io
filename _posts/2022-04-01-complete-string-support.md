---
layout: post
title:  "CrossHair string support, semver, and formal methods"

---

Hello friends. This is another bucket-of-updates kind of newsletter.

I'm hoping to write some more in-depth content soon (things like using CrossHair in practice, what CrossHair v1.0 looks like, a comparison with fuzzing). Let me know what you want to hear about too!


## String support complete

Since v0.0.20, CrossHair has complete string support! CrossHair can reason about every single string method and every single regular expression feature (albeit with varying effectiveness).

Be sure to let me know how it goes if you try it out.


## Contractual SemVer

[SemVer](https://semver.org/) is frequently criticized on the basis that every code change has the potential to break **someone**. We are then challenged to describe what "backwards-compatible" really means.

What if we used contracts to define "backwards-compatible?"

I've proposed a variant of SemVer, [Contractual SemVer](https://github.com/pschanely/contractual-semver), to explore that idea.

CrossHair can help library authors and consumers using Contractual SemVer, too. [See this example](https://crosshair.readthedocs.io/en/latest/case_studies.html#contractual-semver).


## Formal Methods and CrossHair

At present, CrossHair is "bug finder" and not a	verification tool: in particular, your code can easily have bugs that CrossHair does not discover. See [this discussion](https://github.com/pschanely/CrossHair/discussions/156) for details.

However, [Loïc Montandon](https://github.com/lmontand) is actively pushing CrossHair in the verification-tool direction. As a result of his work, CrossHair will:

1. More accurately describe when CrossHair's symbolic exploration has been exhaustive.
2. Produce an exhaustive result	in more	situations.

I am really exicted about this work. Stay tuned for the details.
