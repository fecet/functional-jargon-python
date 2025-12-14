# Functional Programming Jargon

[![Build Status](https://github.com/dry-python/functional-jargon-python/workflows/test/badge.svg?event=push)](https://github.com/dry-python/functional-jargon-python/actions?query=workflow%3Atest)

Functional programming (FP) provides many advantages, and its popularity has been increasing as a result. However, each programming paradigm comes with its own unique jargon and FP is no exception. By providing a glossary, we hope to make learning FP easier.

This is a fork of [Functional Programming Jargon](https://github.com/jmesyou/functional-programming-jargon).

This document is WIP and pull requests are welcome!

__Table of Contents__
<!-- RM(noparent,notop) -->

* [Side effects](#side-effects)
* [Purity](#purity)
* [Idempotent](#idempotent)
* [Arity](#arity)
* [IO](#io)
* [Higher-Order Functions (HOF)](#higher-order-functions-hof)
* [Closure](#closure)
* [Partial Application](#partial-application)
* [Currying](#currying)
* [Function Composition](#function-composition)
* [Continuation](#continuation)
* [Point-Free Style](#point-free-style)
* [Predicate](#predicate)
* [Contracts](#contracts)
* [Category](#category)
* [Value](#value)
* [Constant](#constant)
* [Lift](#lift)
* [Referential Transparency](#referential-transparency)
* [Equational Reasoning](#equational-reasoning)
* [Lambda](#lambda)
* [Lambda Calculus](#lambda-calculus)
* [Lazy evaluation](#lazy-evaluation)
* [Functor](#functor)
* [Applicative Functor](#applicative-functor)
* [Monoid](#monoid)
* [Monad](#monad)
* [Comonad](#comonad)
* [Morphism](#morphism)
  * [Endomorphism](#endomorphism)
  * [Isomorphism](#isomorphism)
  * [Homomorphism](#homomorphism)
  * [Catamorphism](#catamorphism)
  * [Anamorphism](#anamorphism)
  * [Hylomorphism](#hylomorphism)
  * [Paramorphism](#paramorphism)
  * [Apomorphism](#apomorphism)
* [Setoid](#setoid)
* [Semigroup](#semigroup)
* [Foldable](#foldable)
* [Lens](#lens)
* [Type Signatures](#type-signatures)
* [Algebraic data type](#algebraic-data-type)
  * [Sum type](#sum-type)
  * [Product type](#product-type)
* [Option](#option)
* [Function](#function)
* [Partial function](#partial-function)


<!-- /RM -->


## Side effects

A function or expression is said to have a side effect if apart from returning a value, 
it interacts with (reads from or writes to) external mutable state:

```python
>>> print('This is a side effect!')
This is a side effect!
>>>
```

Or:

```python
>>> numbers = []
>>> numbers.append(1)  # mutates the `numbers` array
>>>
```


## Purity

A function is pure if the return value is only determined by its
input values, and does not produce any side effects.

This function is pure:

```python
>>> def add(first: int, second: int) -> int:
...    return first + second
>>>
```

As opposed to each of the following:

```python
>>> def add_and_log(first: int, second: int) -> int:
...    print('Sum is:', first + second)  # print is a side effect
...    return first + second
>>>
```


## Idempotent

A function is idempotent if reapplying it to its result does not produce a different result:

```python
>>> assert sorted([2, 1]) == [1, 2]
>>> assert sorted(sorted([2, 1])) == [1, 2]
>>> assert sorted(sorted(sorted([2, 1]))) == [1, 2]
>>>
```

Or:

```python
>>> assert abs(abs(abs(-1))) == abs(-1)
>>>
```


## Arity

The number of arguments a function takes. From words like unary, binary, ternary, etc. This word has the distinction of being composed of two suffixes, "-ary" and "-ity". Addition, for example, takes two arguments, and so it is defined as a binary function or a function with an arity of two. Such a function may sometimes be called "dyadic" by people who prefer Greek roots to Latin. Likewise, a function that takes a variable number of arguments is called "variadic," whereas a binary function must be given two and only two arguments, currying and partial application notwithstanding.

We can use the `inspect` module to know the arity of a function, see the example below:

```python
>>> from inspect import signature

>>> def multiply(number_one: int, number_two: int) -> int:  # arity 2
...     return number_one * number_two

>>> assert len(signature(multiply).parameters) == 2
>>>
```

### Arity Distinctions

#### Minimum Arity and Maximum Arity

The __minimum arity__ is the smallest number of arguments the function expects to work, the __maximum arity__ is the largest number of arguments function can take. Generally, these numbers are different when our function has default parameter values.

```python
>>> from inspect import getfullargspec
>>> from typing import Any

>>> def example(a: Any, b: Any, c: Any = None) -> None:  # mim arity: 2 | max arity: 3
...     pass

>>> example_args_spec = getfullargspec(example)
>>> max_arity = len(example_args_spec.args)
>>> min_arity = max_arity - len(example_args_spec.defaults)

>>> assert max_arity == 3
>>> assert min_arity == 2
>>>
```

#### Fixed Arity and Variable Arity

A function has __fixed arity__ when you have to call it with the same number of arguments as the number of its parameters and a function has __variable arity__ when you can call it with variable number of arguments, like functions with default parameters values.

```python
>>> from typing import Any

>>> def fixed_arity(a: Any, b: Any) -> None:  # we have to call with 2 arguments
...     pass

>>> def variable_arity(a: Any, b: Any = None) -> None:  # we can call with 1 or 2 arguments
...     pass
>>>
```

#### Definitive Arity and Indefinite Arity

When a function can receive a finite number of arguments it has __definitive arity__, otherwise if the function can receive an undefined number of arguments it has __indefinite arity__. We can reproduce the __indefinite arity__ using Python _*args_ and _**kwargs_, see the example below:

```python
>>> from typing import Any

>>> def definitive_arity(a: Any, b: Any = None) -> None: # we can call just with 1 or 2 arguments
...     pass

>>> def indefinite_arity(*args: Any, **kwargs: Any) -> None: # we can call with how many arguments we want
...     pass
>>>
```

### Arguments vs Parameters

There is a little difference between __arguments__ and __parameters__:

* __arguments__: are the values that are passed to a function
* __parameters__: are the variables in the function definition


## Higher-Order Functions (HOF)

A function that takes a function as an argument and/or returns a function, basically we can treat functions as a value.
In Python every function/method can be a Higher-Order Function.

The functions like `reduce`, `map` and `filter` are good examples of __HOF__, they receive a function as their first argument.
```python
>>> from functools import reduce

>>> reduce(lambda accumulator, number: accumulator + number, [1, 2, 3])
6
>>>
```

We can create our own __HOF__, see the example below:

```python
>>> from typing import Callable

>>> def get_transform_function() -> Callable[[str], int]:
...     return int

>>> def transform[_ValueType, _ReturnType](
...     transform_function: Callable[[_ValueType], _ReturnType],
...     value_to_transform: _ValueType,
... ) -> _ReturnType:
...     return transform_function(value_to_transform)

>>> transform_function = get_transform_function()
>>> assert transform(transform_function, '42') == 42
>>>
```


## IO

IO basically means Input/Output, but it is widely used to just tell that a function is impure.

We have a special type (``IO``) and a decorator (``@impure``) to do that in Python:

```python
>>> import random
>>> from returns.io import IO, impure

>>> @impure
... def get_random_number() -> int:
...     return random.randint(0, 100)

>>> assert isinstance(get_random_number(), IO)
>>>
```

__Further reading__:
* [`IO` and `@impure` docs](https://returns.readthedocs.io/en/latest/pages/io.html)


## Closure

A closure is a way of accessing a variable outside its scope.
Formally, a closure is a technique for implementing lexically scoped named binding. It is a way of storing a function with an environment.

A closure is a scope which captures local variables of a function for access even after the execution has moved out of the block in which it is defined.
ie. they allow referencing a scope after the block in which the variables were declared has finished executing.


```python
>>> def make_multiplier(factor: int):
...     def multiply(value: int) -> int:
...         return value * factor
...     return multiply
...
>>> double = make_multiplier(2)
>>> triple = make_multiplier(3)
>>> assert double(5) == 10
>>> assert triple(4) == 12
```

Lexical scoping is the reason why it is able to find the values of x and add - the private variables of the parent which has finished executing. This value is called a Closure.

The stack along with the lexical scope of the function is stored in form of reference to the parent. This prevents the closure and the underlying variables from being garbage collected(since there is at least one live reference to it).

Lambda Vs Closure: A lambda is essentially a function that is defined inline rather than the standard method of declaring functions. Lambdas can frequently be passed around as objects.

A closure is a function that encloses its surrounding state by referencing fields external to its body. The enclosed state remains across invocations of the closure.

__Further reading/Sources__
* [Lambda Vs Closure](http://stackoverflow.com/questions/220658/what-is-the-difference-between-a-closure-and-a-lambda)
* [JavaScript Closures highly voted discussion](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work)


## Partial Application

Partially applying a function means creating a new function by pre-filling some of the arguments to the original function.
You can also use `functools.partial` or `returns.curry.partial` to partially apply a function in Python:

```python
>>> from returns.curry import partial

>>> def takes_three_arguments(arg1: int, arg2: int, arg3: int) -> int:
...     return arg1 + arg2 + arg3

>>> assert partial(takes_three_arguments, 1, 2)(3) == 6
>>> assert partial(takes_three_arguments, 1)(2, 3) == 6
>>> assert partial(takes_three_arguments, 1, 2, 3)() == 6
>>>
```

The difference between `returns.curry.partial` and `functools.partial` 
is in how types are infered:

```python
import functools

reveal_type(functools.partial(takes_three_arguments, 1))
# Revealed type is 'functools.partial[builtins.int*]'

reveal_type(partial(takes_three_arguments, 1))
# Revealed type is 'def (arg2: builtins.int, arg3: builtins.int) -> builtins.int'
```

Partial application helps create simpler functions from more complex ones by baking in data when you have it. [Curried](#currying) functions are automatically partially applied.

__Further reading__
* [`@curry` docs](https://returns.readthedocs.io/en/latest/pages/curry.html#partial)
* [`functools` docs](https://docs.python.org/3/library/functools.html#functools.partial)


## Currying

The process of converting a function that takes multiple arguments into a function that takes them one at a time.

Each time the function is called it only accepts one argument and returns a function that takes one argument until all arguments are passed.

```python
>>> from returns.curry import curry

>>> @curry
... def takes_three_args(a: int, b: int, c: int) -> int:
...     return a + b + c

>>> assert takes_three_args(1)(2)(3) == 6
>>>
```

Some implementations of curried functions 
can also take several of arguments instead of just a single argument:

```python
>>> assert takes_three_args(1, 2)(3) == 6
>>> assert takes_three_args(1)(2, 3) == 6
>>> assert takes_three_args(1, 2, 3) == 6
>>>
```

Let's see what type `takes_three_args` has to get a better understanding of its features:

```python
reveal_type(takes_three_args)

# Revealed type is:
# Overload(
#   def (a: builtins.int) -> Overload(
#     def (b: builtins.int, c: builtins.int) -> builtins.int, 
#     def (b: builtins.int) -> def (c: builtins.int) -> builtins.int
#   ), 
#   def (a: builtins.int, b: builtins.int) -> def (c: builtins.int) -> builtins.int, 
#   def (a: builtins.int, b: builtins.int, c: builtins.int) -> builtins.int
# )'
```

__Further reading__
* [`@curry` docs](https://returns.readthedocs.io/en/latest/pages/curry.html#id3)
* [Favoring Curry](http://fr.umio.us/favoring-curry/)
* [Hey Underscore, You're Doing It Wrong!](https://www.youtube.com/watch?v=m3svKOdZijA)


## Function Composition

For example, you can compose `abs` and `int` functions like so:

```python
>>> assert abs(int('-1')) == 1
>>>
```

You can also create a third function 
that will have an input of the first one and an output of the second one:

```python
>>> from typing import Callable

>>> def compose[_FirstType, _SecondType, _ThirdType](
...     first: Callable[[_FirstType], _SecondType],
...     second: Callable[[_SecondType], _ThirdType],
... ) -> Callable[[_FirstType], _ThirdType]:
...     return lambda argument: second(first(argument))

>>> assert compose(int, abs)('-1') == 1
>>>
```

We already have this functions defined as `returns.functions.compose`!

```python
>>> from returns.functions import compose
>>> assert compose(bool, str)([]) == 'False'
>>>
```

__Further reading__
* [`compose` docs](https://returns.readthedocs.io/en/latest/pages/functions.html#compose)


## Continuation

At any given point in a program, the part of the code that's yet to be executed is known as a continuation.

```python
>>> from typing import Callable
>>>
>>> def with_continuation[_Value, _Result](
...     value: _Value,
...     continuation: Callable[[_Value], _Result],
... ) -> _Result:
...     return continuation(value)
...
>>> assert with_continuation('hi', str.upper) == 'HI'
```

Continuations are often seen in asynchronous programming when the program needs to wait to receive data before it can continue. The response is often passed off to the rest of the program, which is the continuation, once it's been received.

```python
>>> def greet_async(name: str, callback: Callable[[str], None]) -> None:
...     message = f'Hello, {name}'
...     callback(message)
...
>>> messages = []
>>> greet_async('Jane', messages.append)
>>> assert messages == ['Hello, Jane']
```


## Point-Free Style

Point-Free is a style of writting code without using any intermediate variables.

Basically, you will end up with long chains of direct function calls.
This style usually requires [currying](#currying) or other [Higher-Order functions](#higher-order-functions-hof). 
This technique is also sometimes called "Tacit programming".

The most common example of Point-Free programming style is Unix with pipes:

```bash
ps aux | grep [k]de | gawk '{ print $2 }'
```

It also works for Python, let's say you have this function composition:

```python
>>> str(bool(abs(-1)))
'True'
>>>
```

It might be problematic method methods on the first sight, because you need an instance to call a method on.
But, you can always use HOF to fix that and compose normally:

```python
>>> from returns.pipeline import flow
>>> from returns.pointfree import map_
>>> from returns.result import Success

>>> assert flow(
...     Success(-2),
...     map_(abs),
...     map_(range),
...     map_(list),
... ) == Success([0, 1])
>>>
```

__Further reading:__
* [Pointfree docs](https://returns.readthedocs.io/en/latest/pages/pointfree.html)


## Predicate

A predicate is a function that returns true or false for a given value.
So, basically a predicate is an alias for `Callable[[_ValueType], bool]`.

It is very useful when working with `if`, `all`, `any`, etc.

```python
>>> def is_long(item: str) -> bool:
...     return len(item) > 3

>>> assert all(is_long(item) for item in ['1234', 'abcd'])
>>>
```

__Futher reading__
* [Predicate logic](https://en.wikipedia.org/wiki/Predicate_functor_logic)
* [`cond` docs](https://returns.readthedocs.io/en/latest/pages/pointfree.html#cond)


## Contracts

A contract specifies the obligations and guarantees of the behavior from a function or expression at runtime. This acts as a set of rules that are expected from the input and output of a function or expression, and errors are generally reported whenever a contract is violated.

```python
>>> def positive_only(number: int) -> int:
...     if number <= 0:
...         raise ValueError('Number must be positive')
...     return number
...
>>> assert positive_only(3) == 3
>>> try:
...     positive_only(0)
... except ValueError as error:
...     assert str(error) == 'Number must be positive'
```

## Category

A category in category theory is a collection of objects and morphisms between them. In programming, typically types
act as the objects and functions as morphisms.

To be a valid category 3 rules must be met:

1. There must be an identity morphism that maps an object to itself.
    Where `a` is an object in some category,
    there must be a function from `a -> a`.
2. Morphisms must compose.
    Where `a`, `b`, and `c` are objects in some category,
    and `f` is a morphism from `a -> b`, and `g` is a morphism from `b -> c`;
    `g(f(x))` must be equivalent to `(g • f)(x)`.
3. Composition must be associative
    `f • (g • h)` is the same as `(f • g) • h`

Since these rules govern composition at very abstract level, category theory is great at uncovering new ways of composing things.

```python
>>> from typing import Callable
>>>
>>> def identity[_TypeA](value: _TypeA) -> _TypeA:
...     return value
...
>>> def compose[_TypeA, _TypeB, _TypeC](
...     first: Callable[[_TypeA], _TypeB],
...     second: Callable[[_TypeB], _TypeC],
... ) -> Callable[[_TypeA], _TypeC]:
...     return lambda inner: second(first(inner))
...
>>> to_str = str
>>> length = len
>>> assert compose(identity, to_str)(5) == '5'
>>> assert compose(to_str, length)([1, 2, 3]) == 3
>>> assert compose(identity, compose(to_str, length))([0]) == 1
```

__Further reading__

* [Category Theory for Programmers](https://bartoszmilewski.com/2014/10/28/category-theory-for-programmers-the-preface/)

## Value

Anything that can be assigned to a variable.

```python
>>> number = 7
>>> text = 'hello'
>>> items = [1, 2, 3]
>>> assert number * 2 == 14
>>> assert text.upper() == 'HELLO'
>>> assert sum(items) == 6
```

## Constant

A variable that cannot be reassigned once defined.

```python
>>> from typing import Final
>>>
>>> PI: Final[float] = 3.14159
>>> assert round(PI * 2, 2) == 6.28
```

Constants are [referentially transparent](#referential-transparency). That is, they can be replaced with the values that they represent without affecting the result.

```python
>>> radius = 3
>>> area = PI * radius ** 2
>>> assert area == PI * (radius ** 2)
```

## Lift

Lifting is when you take a value and put it into an object like a [Functor](#functor). If you lift a function into an [Applicative Functor](#applicative-functor) then you can make it work on values that are also in that functor.

Some implementations have a function called `lift`, or `liftA2` to make it easier to run functions on functors.

```python
>>> def lift[_LiftType](value: _LiftType) -> list[_LiftType]:
...     return [value]
...
>>> assert list(map(abs, lift(-2))) == [2]
```

Lifting a one-argument function and applying it does the same thing as `map`.

```python
>>> from typing import Callable, Iterable
>>>
>>> def lift_a2[_Left, _Right, _ResultType](
...     function: Callable[[_Left, _Right], _ResultType],
... ) -> Callable[[Iterable[_Left], Iterable[_Right]], list[_ResultType]]:
...     return lambda first, second: [
...         function(left, right) for left, right in zip(first, second)
...     ]
...
>>> add = lambda left, right: left + right
>>> lifted_add = lift_a2(add)
>>> assert lifted_add([1, 2], [10]) == [11, 12]
```


## Referential Transparency

An expression that can be replaced with its value without changing the
behavior of the program is said to be referentially transparent.

Say we have function greet:

```python
>>> def greet(name: str) -> str:
...     return f'Hello, {name}'
...
>>> greeting = greet('Alice')
>>> assert greeting == 'Hello, Alice'
>>> assert f'{greet("Alice")}!' == f'{greeting}!'
```

## Equational Reasoning

When an application is composed of expressions and devoid of side effects, truths about the system can be derived from the parts.

```python
>>> def square(number: int) -> int:
...     return number * number
...
>>> def double(number: int) -> int:
...     return number * 2
...
>>> assert square(double(3)) == square(6)
>>> assert square(6) == 36
```

## Lambda

An anonymous function that can be treated like a value.

```python
>>> def f(a: int) -> int:
...     return a + 1
...
>>> assert f(1) == (lambda a: a + 1)(1)
```

Lambdas are often passed as arguments to Higher-Order functions.

```python
>>> assert list(map(lambda x: x + 1, [1, 2])) == [2, 3]
```

You can assign a lambda to a variable.

```python
>>> add1 = lambda a: a + 1
>>> assert add1(4) == 5
```

## Lambda Calculus

A branch of mathematics that uses functions to create a [universal model of computation](https://en.wikipedia.org/wiki/Lambda_calculus).

```python
>>> true = lambda a: lambda b: a
>>> false = lambda a: lambda b: b
>>> and_ = lambda first: lambda second: first(second)(first)
>>>
>>> assert and_(true)(false) is false
>>> assert and_(true)(true) is true
```

## Lazy evaluation

Lazy evaluation is a call-by-need evaluation mechanism that delays the evaluation of an expression until its value is needed. In functional languages, this allows for structures like infinite lists, which would not normally be available in an imperative language where the sequencing of commands is significant.

```python
>>> def naturals():
...     current = 0
...     while True:
...         yield current
...         current += 1
...
>>> numbers = naturals()
>>> assert next(numbers) == 0
>>> assert next(numbers) == 1
>>> assert next(numbers) == 2
```

## Functor

An object that implements a `map` method which, while running over each value in the object to produce a new object, adheres to two rules:

__Identity law__

```python
functor.map(lambda x: x) == functor
```

__Associative law__

```python
functor.map(compose(f, g)) == functor.map(g).map(f)
```

Sometimes `Functor` can be called `Mappable` to its `.map` method.
You can have a look at the real-life [`Functor` interface](https://github.com/dry-python/returns/blob/master/returns/interfaces/mappable.py):

```python
>>> from typing import Callable
>>> from returns.interfaces.mappable import Mappable1 as Functor
>>> from returns.primitives.hkt import SupportsKind1

>>> class Box[_FirstType](SupportsKind1['Box', _FirstType], Functor[_FirstType]):
...     def __init__(self, inner_value: _FirstType) -> None:
...         self._inner_value = inner_value
...
...     def map[_NewFirstType](
...         self,
...         function: Callable[[_FirstType], _NewFirstType],
...     ) -> 'Box[_NewFirstType]':
...         return Box(function(self._inner_value))
...
...     def __eq__(self, other) -> bool:
...         return type(other) == type(self) and self._inner_value == other._inner_value

>>> assert Box(-5).map(abs) == Box(5)
>>>
```

__Further reading:__

- [Functor interface docs](https://returns.readthedocs.io/en/latest/pages/interfaces.html#mappable)


## Applicative Functor

An Applicative Functor is an object with `apply` and `.from_value` methods:
- `.apply` applies a function in the object to a value in another object of the same type. Somethimes this method is also called `ap`
- `.from_value` creates a new Applicative Functor from a pure value. Sometimes this method is also called `pure`

All Applicative Functors must also follow [a bunch of laws](https://returns.readthedocs.io/en/latest/pages/interfaces.html#applicative).

__Further reading:__

- [`Applicative Functor` interface docs](https://github.com/dry-python/returns/blob/master/returns/interfaces/applicative.py)


## Monoid

An object with a function that "combines" that object with another of the same type
and an "empty" value, which can be added with no effect.

One simple monoid is the addition of numbers 
(with `__add__` as an addition function and `0` as an empty element):

```python
>>> assert 1 + 1 + 0 == 2
>>>
```

Tuples, lists, and strings are also monoids:

```python
>>> assert (1,) + (2,) + () == (1, 2)
>>> assert [1] + [2] + [] == [1, 2]
>>> assert 'a' + 'b' + '' == 'ab'
>>>
```


## Monad

A monad is an [Applicative Functor](#applicative-functor) with `bind` method. 
`bind` is like [`map`](#functor) except it un-nests the resulting nested object.

```python
>>> from returns.result import Result, Success, Failure
>>>
>>> def safe_divide(dividend: int, divisor: int) -> Result[float, Exception]:
...     if divisor == 0:
...         return Failure(ZeroDivisionError('Cannot divide by zero'))
...     return Success(dividend / divisor)
...
>>> assert Success(10).bind(lambda value: safe_divide(value, 2)) == Success(5.0)
>>> failed_result = Success(10).bind(lambda value: safe_divide(value, 0))
>>> assert isinstance(failed_result, Failure)
```

`of` is also known as `return` in other functional languages.
`chain` is also known as `flatmap` and `bind` in other languages.

## Comonad

An object that has `extract` and `extend` functions.

```python
>>> from typing import Callable, Generic
>>>
>>> class CoIdentity[_CoValue](Generic[_CoValue]):
...     def __init__(self, value: _CoValue) -> None:
...         self._value = value
...
...     def extract(self) -> _CoValue:
...         return self._value
...
...     def extend[_NewCoValue](
...         self,
...         function: Callable[['CoIdentity[_CoValue]'], _NewCoValue],
...     ) -> 'CoIdentity[_NewCoValue]':
...         return CoIdentity(function(self))
...
>>> square_context = CoIdentity(5).extend(lambda context: context.extract() ** 2)
>>> assert square_context.extract() == 25
```

## Morphism

A transformation function.

### Endomorphism

A function where the input type is the same as the output.

```python
>>> uppercase = lambda text: text.upper()
>>> decrement = lambda number: number - 1
>>> assert uppercase('hi') == 'HI'
>>> assert decrement(4) == 3
```

### Isomorphism

A pair of transformations between 2 types of objects that is structural in nature and no data is lost.

```python
>>> def to_bytes(text: str) -> bytes:
...     return text.encode('utf-8')
...
>>> def from_bytes(raw: bytes) -> str:
...     return raw.decode('utf-8')
...
>>> assert from_bytes(to_bytes('hello')) == 'hello'
```

### Homomorphism

A homomorphism is just a structure preserving map. In fact, a functor is just a homomorphism between categories as it preserves the original category's structure under the mapping.

```python
>>> numbers = [1, 2, 3]
>>> as_tuple = tuple(numbers)
>>> assert as_tuple == (1, 2, 3)
```

### Catamorphism

A `reduce_right` function that applies a function against an accumulator and each value of the array (from right-to-left) to reduce it to a single value.

```python
>>> from typing import Callable, Iterable
>>>
>>> def fold_right[_FoldType, _AccType](
...     items: Iterable[_FoldType],
...     initial: _AccType,
...     function: Callable[[_FoldType, _AccType], _AccType],
... ) -> _AccType:
...     result = initial
...     for item in reversed(list(items)):
...         result = function(item, result)
...     return result
...
>>> assert fold_right([1, 2, 3], 0, lambda item, acc: item + acc) == 6
```

### Anamorphism

An `unfold` function. An `unfold` is the opposite of `fold` (`reduce`). It generates a list from a single value.

```python
>>> from typing import Callable
>>>
>>> def unfold(seed: int, stop: Callable[[int], bool]) -> list[int]:
...     values = []
...     current = seed
...     while not stop(current):
...         values.append(current)
...         current += 1
...     return values
...
>>> assert unfold(1, lambda value: value > 3) == [1, 2, 3]
```

### Hylomorphism

The combination of anamorphism and catamorphism.

```python
>>> assert fold_right(unfold(1, lambda value: value > 3), 0, lambda item, acc: item + acc) == 6
```

### Paramorphism

A function just like `reduce_right`. However, there's a difference:

In paramorphism, your reducer's arguments are the current value, the reduction of all previous values, and the list of values that formed that reduction.

```python
>>> from typing import Callable
>>>
>>> def para(
...     items: list[int],
...     initial: int,
...     function: Callable[[int, list[int], int], int],
... ) -> int:
...     if not items:
...         return initial
...     head, *tail = items
...     return function(head, tail, para(tail, initial, function))
...
>>> assert para([1, 2, 3], 0, lambda current, rest, acc: acc + current + len(rest)) == 9
```

### Apomorphism

it's the opposite of paramorphism, just as anamorphism is the opposite of catamorphism. Whereas with paramorphism, you combine with access to the accumulator and what has been accumulated, apomorphism lets you `unfold` with the potential to return early.

```python
>>> from typing import Callable
>>>
>>> def apo(seed: int, step: Callable[[int], tuple[int, int | None]]):
...     result = []
...     current = seed
...     while current is not None:
...         value, current = step(current)
...         result.append(value)
...     return result
...
>>> assert apo(0, lambda number: (number, None if number >= 2 else number + 1)) == [0, 1, 2]
```

## Setoid

An object that has an `equals` function which can be used to compare other objects of the same type.

Make array a setoid:

```python 
>>> class Point:
...     def __init__(self, x: int, y: int) -> None:
...         self.x = x
...         self.y = y
...
...     def equals(self, other: 'Point') -> bool:
...         return self.x == other.x and self.y == other.y
...
>>> assert Point(1, 2).equals(Point(1, 2))
>>> assert not Point(1, 2).equals(Point(2, 1))
```

## Semigroup

An object that has a `concat` function that combines it with another object of the same type.

```python
>>> def concat_numbers(first: int, second: int) -> int:
...     return first + second
...
>>> assert concat_numbers(1, 2) == 3
>>> assert concat_numbers(concat_numbers(1, 2), 3) == concat_numbers(1, concat_numbers(2, 3))
```

## Foldable

An object that has a `reduce` function that applies a function against an accumulator and each element in the array (from left to right) to reduce it to a single value.

```python
>>> from functools import reduce
>>> assert reduce(lambda acc, value: acc + value, [1, 2, 3], 0) == 6
```

## Lens

A lens is a structure (often an object or function) that pairs a getter and a non-mutating setter for some other data
structure.

```python
>>> def lens(key):
...     return (
...         lambda data: data[key],
...         lambda value, data: {**data, key: value},
...     )
...
>>> get_name, set_name = lens('name')
>>> person = {'name': 'Jane', 'age': 30}
>>> assert get_name(person) == 'Jane'
>>> assert set_name('John', person) == {'name': 'John', 'age': 30}
```

Lenses are also composable. This allows easy immutable updates to deeply nested data.

```python
>>> address_lens = lens('address')
>>> city_lens = lens('city')
>>>
>>> def compose_lens(left, right):
...     left_get, left_set = left
...     right_get, right_set = right
...     return (
...         lambda data: right_get(left_get(data)),
...         lambda value, data: left_set(right_set(value, left_get(data)), data),
...     )
...
>>> address_city_lens = compose_lens(address_lens, city_lens)
>>> person_with_address = {'name': 'Jane', 'address': {'city': 'Oslo'}}
>>> composed_get, composed_set = address_city_lens
>>> assert composed_get(person_with_address) == 'Oslo'
>>> assert composed_set('Berlin', person_with_address) == {
...     'name': 'Jane',
...     'address': {'city': 'Berlin'},
... }
```

## Type Signatures

Type signatures describe the types a function accepts and returns.

```python
>>> def head(items: list[int]) -> int:
...     return items[0]
...
>>> assert head([1, 2, 3]) == 1
```

__Further reading__
* [Ramda's type signatures](https://github.com/ramda/ramda/wiki/Type-Signatures)
* [Mostly Adequate Guide](https://drboolean.gitbooks.io/mostly-adequate-guide/content/ch7.html#whats-your-type)
* [What is Hindley-Milner?](http://stackoverflow.com/a/399392/22425) on Stack Overflow

## Algebraic data type

A composite type made from putting other types together. Two common classes of algebraic types are [sum](#sum-type) and [product](#product-type).

### Sum type

A Sum type is the combination of two types together into another one. It is called sum because the number of possible values in the result type is the sum of the input types.

```python
>>> from dataclasses import dataclass
>>> from typing import Union
>>>
>>> @dataclass
... class Success:
...     value: int
...
>>> @dataclass
... class Failure:
...     message: str
...
>>> Response = Union[Success, Failure]
>>>
>>> def render(response: Response) -> str:
...     if isinstance(response, Success):
...         return f'value: {response.value}'
...     return f'error: {response.message}'
...
>>> assert render(Success(2)) == 'value: 2'
>>> assert render(Failure('no data')) == 'error: no data'
```

Sum types are sometimes called union types, discriminated unions, or tagged unions.

The [sumtypes](https://github.com/radix/sumtypes/) library in Python helps with defining and using union types.

### Product type

A __product__ type combines types together in a way you're probably more familiar with:

```python
>>> from dataclasses import dataclass
>>>
>>> @dataclass
... class User:
...     username: str
...     active: bool
...
>>> assert User('alice', True).username == 'alice'
```

See also [Set theory](https://en.wikipedia.org/wiki/Set_theory).

## Option

Option is a [sum type](#sum-type) with two cases often called `Some` and `None`.

Option is useful for composing functions that might not return a value.

```python
>>> from returns.maybe import Maybe, Nothing, Some
>>>
>>> def safe_head(items: list[int]) -> Maybe[int]:
...     if not items:
...         return Nothing
...     return Some(items[0])
...
>>> assert safe_head([1, 2, 3]) == Some(1)
>>> assert safe_head([]) is Nothing
```

`Option` is also known as `Maybe`. `Some` is sometimes called `Just`. `None` is sometimes called `Nothing`.

## Function

A __function__ `f :: A => B` is an expression - often called arrow or lambda expression - with __exactly one (immutable)__ parameter of type `A` and __exactly one__ return value of type `B`. That value depends entirely on the argument, making functions context-independent, or [referentially transparent](#referential-transparency). What is implied here is that a function must not produce any hidden [side effects](#side-effects) - a function is always [pure](#purity), by definition. These properties make functions pleasant to work with: they are entirely deterministic and therefore predictable. Functions enable working with code as data, abstracting over behaviour:

```python
>>> from typing import Callable
>>>
>>> def add_one(value: int) -> int:
...     return value + 1
...
>>> def apply(function: Callable[[int], int], value: int) -> int:
...     return function(value)
...
>>> assert apply(add_one, 1) == 2
```

## Partial function

A partial function is a [function](#function) which is not defined for all arguments - it might return an unexpected result or may never terminate. Partial functions add cognitive overhead, they are harder to reason about and can lead to runtime errors. Some examples:

```python
>>> def head(items: list[int]) -> int:
...     return items[0]
...
>>> assert head([1, 2, 3]) == 1
>>> try:
...     head([])
... except IndexError:
...     pass
```

### Dealing with partial functions

Partial functions are dangerous as they need to be treated with great caution. You might get an unexpected (wrong) result or run into runtime errors. Sometimes a partial function might not return at all. Being aware of and treating all these edge cases accordingly can become very tedious.
Fortunately a partial function can be converted to a regular (or total) one. We can provide default values or use guards to deal with inputs for which the (previously) partial function is undefined. Utilizing the [`Option`](#option) type, we can yield either `Some(value)` or `None` where we would otherwise have behaved unexpectedly:

```python
>>> from returns.maybe import Maybe, Nothing, Some
>>>
>>> def safe_head(items: list[int]) -> Maybe[int]:
...     if not items:
...         return Nothing
...     return Some(items[0])
...
>>> assert safe_head([1, 2, 3]) == Some(1)
>>> assert safe_head([]) is Nothing
```
