---
layout: post
title:  "Contracts Propagate Requirements"

---

[Contracts](https://en.wikipedia.org/wiki/Design_by_contract) and
[property-based testing](https://increment.com/testing/in-praise-of-property-based-testing/)
have a lot of overlap: they both can be used to check arbitrary behaviors of your code.
But contracts **propagate**, and that has big implications.


I was very happy to share a
[EuroPython talk](https://ep2021.europython.eu/talks/5UJcSrs-automatic-testing-of-python-functions-based-on-contracts/)
with
[Marko Ristin-Kaufmann](https://github.com/mristin)
and
[Lauren De bruyn](https://www.linkedin.com/in/lauren-de-bruyn) today.
We introduced the concept of code contracts and two tools that can be used to check contracts,
[icontract-hypothesis](https://github.com/mristin/icontract-hypothesis)
and
[CrossHair](https://github.com/pschanely/CrossHair).

One obvious difference between contracts and property-based tests:
only contracts can be run in staging/production.

But I emphasized another important difference:

> Unlike proprety-based tests, contract requirements **propagate** through your codebase.

I'll explain with the example I used in my talk.
Imagine we're building an online shopping site, and need a function to compute the total
price for an order:


```
class LineItem:
    item_id: str
    quantity: int

def compute_total(items: List[LineItem], prices: Dict[str, float]) -> float:
    total = 0.0
    for item in items:
        total += prices[item.item_id] * item.quantity
    return total
```

One might imagine that we want every order total to be greater than zero.
It's easy enough to make a postcondition for that (in the 
[icontract]()
syntax):

```
@ensure(lambda result: result > 0)
def compute_total(items: List[LineItem], prices: Dict[str, float]) -> float:
    ...
```

[CrossHair]()
quickly points out **several** ways to break this postcondition.

We might dutifully handle each of these cases. There are 4 of them!:

```
# There is at least one item:
@require(lambda items: len(items) > 0)

# Each quantity is at least one:
@require(lambda items: all(i.quantity > 0 for i in items))

# Every item has a price:
@require(lambda items, prices: all(i.item_id in prices for i in items))

# Every price is greater than zero:
@require(lambda prices: all(p > 0 for p in prices.values()))
```

After we add all these preconditions, CrossHair is satisfied.

This is an awful lot of work, and we didn't find a single bug in our function.
Was it worth it?

If this were a property-based test, probably not.

But as a contract, these new preconditions propagate to callers.
And, via more contracts, to the callers of callers.
This kind of propagation is very similar to what happens when you change a type:
the change ripples throughout the system, causing you to notice all the other
places that need to be changed.

Zooming out to a wider scale, each of the preconditions imply requirements for
other parts of the system. Let's look at each of these preconditions again, this
time while "zoomed out":

&nbsp;

```
@require(lambda items: len(items) > 0)
```
You cannot check out if your cart is empty!

&nbsp;

```
@require(lambda items: all(i.quantity > 0 for i in items))
```
Setting a quantity to zero should remove the entire line item!

&nbsp;

```
@require(lambda items, prices: all(i.item_id in prices for i in items))
```
You can't add something to a cart that doesn't have a price!

&nbsp;

```
@require(lambda prices: all(p > 0 for p in prices.values()))
```
When parsing prices from a feed or table, verify the prices are nonzero!

&nbsp;

All of these requirements are important and could be easy to forget.
Contracts, along with tooling like icontract-hypothesis and CrossHair,
help us **discover** them.

Sometimes, we don't want to propagate requirements, and that's fine too.
This comes back to a point that Marko made today - all these testing
approaches (unit tests, property-based tests, contract) are **complementary**.

UPDATE 2021-10-12: The talk recording has been released!
You can jump to the CrossHair part
[here](https://youtu.be/ynRdJR5UQWY?t=1428)!
