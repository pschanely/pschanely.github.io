---
layout: post
title:  "CrossHair unicode and more"

---

Plenty of things to talk about in this update. I'll get right to it.


## Full Unicode Support

In version 0.0.18, CrossHair can generate counterexample strings in full unicode.
Until now, unicode support was probably the most glaring omission in CrossHair's
capability list.

It also has expanded symbolic support for regexes and various string methods.
([full release notes](https://crosshair.readthedocs.io/en/latest/changelog.html#version-0-0-18))


## Deal Support

CrossHair can now check contracts written in
[deal](https://deal.readthedocs.io/)!
Check out
[the details here](file:///Users/pschanely/proj/CrossHair/doc/build/kinds_of_contracts.html#analysis-kind-deal).

One neat thing about deal in particular: it encourages you to tag side-effects of your
functions.
CrossHair uses some of these tags to avoid unintended side effects while checking
contracts.


## EuroPython Talk - Released!

All of the EuroPython recordings
[have been released](https://www.youtube.com/playlist?list=PL8uoeex94UhFuRtXhkqOrROsdNI6ejuiq)!

I shared a talk about contracts, and you can skip ahead to my
[CrossHair section](https://youtu.be/ynRdJR5UQWY?t=1428) if you like.

One of the neat things about this section of the talk is that I get into some of the
[design consequences of developing with contracts specifically](/2021/07/30/contracts-propogate-expectations.html)
(as opposed to related strategies like fuzzing and property-based testing, which 
can also be CrossHair-facilitated).


## Hacktoberfest

CrossHair has
[a few open issues](https://github.com/pschanely/CrossHair/issues?q=is%3Aissue+is%3Aopen+label%3AHacktoberfest)
marked for
[Hacktoberfest](https://hacktoberfest.digitalocean.com/).

That said, one of the biggest ways to help CrossHair is to:
1. Try using CrossHair while out while you work on **other** Hacktoberfest projects!
2. Find CrossHair bugs. File issues.
3. File a pull request against CrossHair with failing unit test(s) 
   (with @pytest.mark.skip).
4. Watch me joyously merge them and get your PR count even higher.


## The Feels

Watching Python take the leader position this month in the
[TIOBE index](https://www.tiobe.com/tiobe-index/), I am filled with optimism.
Python, the language, has some properties that are particularly suited to 
the strengths of SMT solvers. (arbitrary size integers, for one)

We just need to wield that strength.

Let's get to work.
