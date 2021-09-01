---
layout: post
title:  "An early Hypothesis-CrossHair integration"

---

**TLDR**: As of today,
[CrossHair](https://github.com/pschanely/CrossHair)
can check
[Hypothesis](https://hypothesis.readthedocs.io)
tests. (but it's bad at it right now!)

**Preliminary Concepts:**

* [Hypothesis](https://hypothesis.readthedocs.io/) - Write property-based tests in Python
* [CrossHair](https://github.com/pschanely/CrossHair) - Check Python contracts with concolic testing
* [design by contract](https://en.wikipedia.org/wiki/Design_by_contract) - Describe your functions with preconditions and postconditions
* [property-based testing](https://en.wikipedia.org/wiki/Property_testing) - Test properties of your code, not just examples
* [concolic testing](https://en.wikipedia.org/wiki/Concolic_testing) - Test concrete execution paths with symbolic inputs
* [SMT solvers](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories) - Find values that satisfy constraints

## The Story

I
[recently mentioned]({% post_url 2021-07-30-contracts-propogate-expectations %})
that property-based testing and contracts are similar.

This similarity has prompted a variety of great thought and discussions about whether
CrossHair's concolic execution engine could be applied to Hypothesis, including
[this longstanding github issue](https://github.com/pschanely/CrossHair/issues/45).

The idea of applying both fuzzing and symbolic approaches to the same set of
specifications isn't new, though these tend to be bleeding-edge projects
(proptest in Rust can be [both fuzzed and sometimes verified](https://hypofuzz.com/docs/literature.html#rust-proptest-propfuzz-propverify))!

Late last year, 
[Zac Hatfield-Dodds](https://zhd.dev/)
and I brainstormed about this and
[dug into the challenges](https://gist.github.com/Zac-HD/28a81c7fc9c73637eba4b2f675bb3b5a#write-for-hypothesis-run-with-crosshair).
This year,
[Matthew Law](https://github.com/JinghanCode)
experimented with a
[hypothesis-strategy-introspection](https://github.com/JinghanCode/Crosshair-Hypothesis/pull/1)
approach, which I'll touch on below.
And today, I've released a comprehensive-but-slow proof of concept (mostly in commits
[1](https://github.com/pschanely/CrossHair/commit/fd259dc99d2a544f48b83c8ca3647fd39866d2b8),
[2](https://github.com/pschanely/CrossHair/commit/687b791514d489f9d171aaebd17483a1691a0c59), and
[3](https://github.com/pschanely/CrossHair/commit/2bd3b29ef4bb1e303ef47b9f9775c44bfb8064c2)).


## Show it to me!

Sometimes, Hypothesis doesn't pick the right inputs for your test; an example:

```
from hypothesis import given
import hypothesis.strategies as st

def round_to_millions(number):
    return ((number + 500_000) // 1_000_000) * 1_000_000

@given(st.integers())
def test_round(number):
    difference = abs(number - round_to_millions(number))
    assert difference < 500_000  # <- This should be "<=", not "<"
```

Hypothesis doesn't readily guess that it needs to try an number like 500_000 to make
this test fail.

But now you can run CrossHair on the same test file:

```
$ crosshair watch --analysis_kind=hypothesis test_rounding.py
```

And, after almost 3 minutes(!), it'll find a counterexample:
```
I found an exception while running your function.
test_rounding.py:12:
|@given(st.integers())
|def test_round(number):
>    assert abs(number - round_to_millions(number)) < 500_000

AssertionError: 
when calling test_round(number = -500000)
```

Three minutes seems like a long time for a constraint solver to find this input!

In fact, CrossHair's
[contract-based version of the same problem](https://crosshair-web.org/?crosshair=0.1&python=3.8&gist=633aa35fe921e85e310bc431330a4781)
can be solved in seconds. What gives?

CrossHair is symbolically executing all of the hypothesis code which generates function
inputs.
That code is pretty sophisticated - it's essentially parsing values out of a byte
sequence.
So CrossHair needs to do a lot of work before it even gets to the body of the test.

Some datatypes are harder to generate than others;
for example, CrossHair cannot analyze Hypothesis string inputs in any reasonable
timeframe.

## What's Next?

Matthew's work above can help us detect some common cases when we don't need to run the
hypothesis input generation code.
I think this makes a lot of sense in cases where this doesn't add too much complexity
to CrossHair or dependencies on Hypothesis internals - I'm hoping to incorporate at
least some of this work soon.

A more ambitious arc is to introduce a new "mid-level" representation into Hypothesis
itself!
We'd add a layer above the byte string that can generate a fixed set of richer types:
ints, strings, and floats.

The mid-level would have bi-directional transformations with the byte string format.
Then, CrossHair can start at the mid-level representation;
this lets us more directly apply a lot of the type-specific reasoning that SMT solvers
are known for.

Zac
[has some ideas](https://gist.github.com/Zac-HD/28a81c7fc9c73637eba4b2f675bb3b5a#mir-design-notes)
about how this mid-level representation could be added to Hypothesis.
If you're interested in helping to bring more effective symbolic reasoning to
Hypothesis, we'd love your help - don't hesitate to reach out!


[Discuss](https://news.ycombinator.com/item?id=28375398) on HN.