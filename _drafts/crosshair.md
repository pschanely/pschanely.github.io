

<pre>
</pre>

<pre>
</pre>

<pre>
</pre>

avg function

grade_score function

grade gist https://crosshair-web.org/?crosshair=0.1&python=3.8&gist=630d38da23d6c88d94446a914a5768ff

def grade(score: float) -> str:
    '''
    pre: 0 <= score <= 100
    post: len(_) == 1
    post: implies(score < 60, _ == 'F')
    post: implies(50 <= score < 90, chr(ord(_) + 1) == grade(score + 10))
    '''
    if score >= .9:
        return 'A'
    if score >= .8:
        return 'B'
    if score >= .7:
        return 'C'
    if score >= .6:
        return 'D'
    return 'F'


## o


* Logic implemented in C.
* __Semantic differences__ between Python operations and their most similar
  Z3 operations are plentiful. For example, booleans *are*
  integers in Python; not so in Z3:
```
>>> True + True
2
>>> z3.Bool('b') + True
TypeError: unsupported operand type(s) for +: 'BoolRef' and 'bool'
```
  There are tons of differences like this. Negative list indicies. Rounding and
  mod behaviors. Regex greediness. Most of CrossHair's logic is focused on
  reconciling these behaviors.
* __Aliasing__: Z3 values are immutable. Many Python values aren't.
  When your list hold non-atomic types, it is really holding a Z3 value with the
  uninterpreted `HeapRef` sort.
  That `HeapRef` value can be looked up in heap that associates `HeapRef`
  instances with other (symbolic or concrete) Python values.


```
from crosshair.libimpl.builtinslib import SmtInt
SmtInt._ch_smt_sort()
import z3
SmtInt._ch_smt_sort() == z3.IntSort()
SmtInt('x').var
from crosshair.statespace import StateSpace
dir(StateSpace)
from crosshair.statespace import StateSpaceContext

from crosshair.statespace import SinglePathNode
space = StateSpace(5, 5, SinglePathNode())
root = SinglePathNode()
root = SinglePathNode(True)
>>> SmtInt._ch_smt_sort() == z3.IntSort()
True
space = StateSpace(5, 5, root)
ContextStateSpace(space).__enter__()
StateSpaceContext(space).__enter__()
crosshair_x = SmtInt('x')
crosshair_x.var
type(crosshair_x.var)

>>> SmtInt._ch_smt_sort() == z3.IntSort()
True
>>> SmtInt('x').var.sort()

```





When stuff happens to those special objects, we consult the SMT solver and then produce other special objects, and the function continues.

The statespace is a thread local that's created for each execution of the analyzed function; it serves a few purposes that are central to how CrossHair works, but the most important two are:
(1) It stores the current set of z3 constraints for this execution. (it holds a z3.Solver)
(2) It consults and updates an execution branch tree. This tree represents the different execution paths that the analyzed function could take. It uses this tree to avoid taking branches that it's already explored on prior executions and updates the tree with the novel parts of its own execution.

The statepsace almost doesn't need to be a thread-local (and wasn't until recently), but there is a case or two where it's challenging to get at it. One problem is that since CrossHair actually calls the function under analysis, you may find yourself in part of the stack where you can't easily get at the statespace. (imagine that you need to invent a new symbolic value out of nowhere, for instance) Furthermore, it would always be an error to work with values from different statespaces, so we aren't really losing a lot of flexibility by making it thread-local.


the constructor of all Smt* types take a first argument of type `Union[str, z3.ExprRef]`.

When you pass a string, you are naming a fresh z3 variable (all z3 variables must have names). This is how we create, say, an unconstrained  integer to pass into a user function.

That distinction is super confusing here. `SmtStr("0")` is actually creating a new symbolic string with an arbitrary, unknown value and having the **name** "0".  Instead, you could do `SmtStr(z3.StringVal("0"))`.

That said, the `*` operator is overridden by SmtInt (via `__rmul__`) and already does the symbolic work you need ([check it out here, for fun](https://github.com/pschanely/CrossHair/blob/94df30a5a4dd3a7956b34909dfc2ce9f2179eb71/crosshair/libimpl/builtinslib.py#L616-L626)); you might be able to just say `buffer = "0" * offset`.

Taking all my suggestions together, you may be surprised, saying "wait, I'm just writing an implementation of zfill in regular old python - I'm not using z3 at all!" Indeed, a lot of the CrossHair codebase is just re-implementing the python library in regular old python (instead of the C-based implementation that it ships with). We just take special care to ensure we're always building on top of python that we already symbolically handle. This is a good way to start, but I'm happy to help you find something next that will require digging into the solver!



```
class SmtFloat():
    ...
    
    def hex(self) -> str:
        return realize(self).hex()

    def is_integer(self) -> SmtBool:
        return SmtBool(z3.IsInt(self.var))

    def as_integer_ratio(self) -> Tuple[Integral, Integral]:
        space = context_statespace()
        numerator = SmtInt('numerator' + space.uniq())
        denominator = SmtInt('denominator' + space.uniq())
        space.add(denominator.var > 0)
        space.add(numerator.var == denominator.var * self.var)
        # There are many valid integer ratios to return. Experimentally, both
        # z3 and CPython tend to pick the same ones. But verify this, while
        # deferring materialization:
        def ratio_is_chosen_by_cpython() -> bool:
            return realize(self).as_integer_ratio() == (numerator, denominator)
        space.defer_assumption('float.as_integer_ratio gets the right ratio',
                               ratio_is_chosen_by_cpython)
        return (numerator, denominator)

```


AbcString just helps us detect when we hit operations that we don't support; the actual class you want is [`crosshair.libimpl.builtinslib.SmtStr`](https://github.com/pschanely/CrossHair/blob/2a124c7d599c42c95aeb66437ad6e3e62e45b8c1/crosshair/libimpl/builtinslib.py#L1553) (which extends AbcString!).

I've started [a debugging guide](https://gist.github.com/pschanely/0ad5b3b1320ae5aaa2a72ac384322df8) with str.zfill as an example (which might be a good one for you to try fixing!). Maybe see if you can reproduce that to start.

Then, maybe take a look at the [StringsTest](https://github.com/pschanely/CrossHair/blob/2a124c7d599c42c95aeb66437ad6e3e62e45b8c1/crosshair/libimpl/builtinslib_test.py#L290) tests in builtinslib_test.py. Maybe we start with a test like:

```
    def test_zfill_fail(self) -> None:
        def f(s: str) -> str:
            ''' post: __return__ != "00012"'''
            return s.zfill(5)
        self.assertEqual(*check_fail(f))
```

`check_fail` means we expect a counterexample. (but we don't get one until you fix SmtStr somehow!) There are a ton of tests here, so you'll want to run your test individually; something like:

```
python crosshair/libimpl/builtinslib_test.py StringsTest.test_zfill_fail -v
```

I suspect you can fix this by re-implementing zfill() on SmtStr in terms of other string operations that already work, like string concatenation ("a" + "b") and string repetition ("a" * 3). Other string methods might require you to dig into the SMT/z3 stuff more directly. Feel free to reach out when you're ready to implement.

When you're ready to run all the tests (this will take a long time), run `python -m unittest discover -p '*_test.py'`.




A few tempting things that I recommend you avoid: (1) quantifiers, that is
`z3.ForAll` and `z3.Exists` - these tend to quickly drive Z3 to
"unknown satisfiability" which essentially causes CrossHair to give up on the
current execution iteration  (2) function definitions in Z3 (if you find
yourself wanting a Z3 function, try to instead create a regular python function
that generates constraints for Z3 just-in-time). Having a lot of constraints is
sometimes fine; much more important is the kind of constraint. Just as
background, the word "[logic](http://smtlib.cs.uiowa.edu/logics.shtml)" is used
in SMT to describe kinds of constraints that are effectively solvable.
