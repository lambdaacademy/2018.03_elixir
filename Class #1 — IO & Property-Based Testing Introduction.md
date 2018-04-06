# School of Elixir — Class #1 Notes

This year's school is about Property-Based Testing because:

- It's fun

- It's instructive

- It's flexible

But what _is_ property-based testing?

## Property-Based Testing

To see what property-based testing is, compare and contrast the following test cases for Elixir's absolute function.
The next section explains a lot of the syntax.

A traditional test case:

```elixir
test "abs/1 is positive (or zero) regardless of input's parity" do
  table = [{-3,3}, {0,0}, {+5,5}]
  for {input, output} <- table do
    assert abs(input) === output
  end
end
```

A property-based test case:

```elixir
property "abs/1 is positive (or zero) regardless of input's parity" do
  check all i <- integer() do
    assert abs(i) >= 0
  end
end
```

In property-based testing we don't make test data explicit (e.g. the `[{-3,3}, {0,0}, {+5,5}]` above) but instead make it implicit with a generator (the `integer()` function).
We're removing ourselves from making the rote choices for what input to test with.
These choices are down to the programmer who makes arbitrary, if representative, choices.
Whereas we could assert that we get a particular output in the more traditional example, `abs(-3)` gives `+3`, we can no longer do this with the value from the generator.
We should, or have to, assert a more _general_ property.
This is the one imperative to property-based testing.
We can't force the a number to be positive but we can test for it.

The property-based test case represents its intent better than the traditional test case.
The property is made to run many times with different input by the test framework so that we would test `abs` with negative and positive numbers and possibly zero.
The default number of runs is 100.

The challenge here is to write properties that fully capture what it means for `abs` to be correct.
Testing that `abs` returns a positive number doesn't tell us if its input value and output value have the same magnitude.
Can you think of a property that would test this?

## How to do I/O (or how not to do it 😀)

The FizzBuzz exercise in class #1 aimed to make a distinction between purity and I/O:

- Purity means that a function's only effect is to compute its result (Hughes)
- I/O is something that happens outside of a function (reading input or writing output)

```elixir
#! /usr/bin/env elixir

defmodule FizzBuzz do
  def stringify(i) do
    cond do
      rem(i, 3) === 0 and rem(i, 5) === 0 ->
        "FizzBuzz"

      rem(i, 3) === 0 ->
        "Fizz"

      rem(i, 5) === 0 ->
        "Buzz"

      true ->
        Integer.to_string(i)
    end
  end
end

1..100
|> Enum.map(&FizzBuzz.stringify/1)
|> Enum.intersperse("\n")
|> IO.puts()
```

The above is a listing of the code we wrote in class.
One of its good qualities is that `stringify` has a simple signature:
it consumes an integer and produces a string.

## Line-by-line Walkthrough

It might help to have the code above side-by-side with the following points.

- File names are snake-cased in Elixir.
The above resides in a file called `fizz_buzz.exs`.

- The Elixir file extensions are `.ex` for source and `.exs` for scripts.

- `defmodule` introduces a module.
This is the construct where code resides in Elixir.

- Module names introduce a name-space.
These are camel-cased as with `FizzBuzz`.

- `def` introduces a public function and `defp` introduces a private function (but only in the scope of a module).
I.e. named functions, like `stringify`, are public-to or private-to a module.

- Blocks are opened and closed with the `do` and `end` words respectively.
They're the equivalent of C's open brace `{` and close brace `}` or Python's colon `:` and indentation.
In Elixir function definitions aren't the only construct that takes a block of code:
as you've seen `defmodule`, `test`, and `property` also need a block.
These aren't unlike C's or Python's conditional constructs like the `if`, `while`, `for`, or even the `class` which take blocks too.

- Elixir's `cond` is like an `if/else` or `switch` (with a `break`) in other languages.

- The `===` is a strict equality operator.
I.e. it treats integers and floats differently.
Whereas `1 == 1.0` is `true` `1 === 1.0` is `false`.
It checks that the type is the same (integer or float) and then it checks that the value is the same.

- Conditions in a `cond` block (e.g. `rem(i, 3) === 0`) are followed by a `->` which indicates the body of code to evaluate if that condition is `true`.

- Strings are between double quotes in Elixir:
``"This is a string!"`` but `'This isn't!'`.

- There is no explicit return statement in either the function block or the `cond` expression.
The last thing in an expression is its value.

- A condition, the arrow, and a block of code following the arrow constitute a "clause".

- We use a final condition of `true` as a default clause.

- Function names are snake-cased as is `to_string`.

- `1..100` is called a "range".
It represents the numbers from 1 to 100 inclusive.

- The **pipe operator** `|>` has a simple meaning.
  The follow three are equivalent.

  This is good old function application:

  ```Elixir
  Enum.each(Enum.map(1..100, &FizzBuzz.stringify/1), &IO.puts/1)
  ```

  To make it more readable we might write something like the following:

  ```Elixir
  alpha = 1..100
  beta  = Enum.map(alpha, &FizzBuzz.stringify/1)
  gamma = Enum.each(beta, &IO.puts/1)
  ```

  This has more of a concatenative style to it:

  ```Elixir
  1..100
  |> Enum.map(&FizzBuzz.stringify/1)
  |> Enum.each(&IO.puts/1)
  ```

  The pipe implicitly passes the value preceding it as the first argument of the function following it and returns as normal.
  It's like chaining in OOP except that every value between the pipe is not mutable.
  When used properly it can make code more readable.

- The `Enum` module has functions for working with collections like the range `1..100`.
`1..100` is in fact a stream (similar to a generator in Python if you're familiar with those).
A more common collection is the list.
The list literal opens with an opening bracket and closes with a closing bracket to enclose elements: `[0,1,2]`.
Though they are drastically different inside you can think of the stream `1..10` as the list `[1,2,3,4,5,6,7,8,9,10]`.
**Class #2 explores the difference.**

- A **higher-order function** (HOF) is a function that consumes a function as input or produces a function as output or both.
  We'll see these a lot (class #3 will explore this in full).
  This simplest HOF is `apply`.
  Its signature in Elixir is:

  `apply(f :: function(), arguments :: list())`

  We might use `apply` like so:

  ```Elixir
  apply(f, [1, "two", 'three'])
  ```

  This has the same effect as:

  ```Elixir
  f(1, "two", 'three')
  ```

- We write `FizzBuzz.stringify/1` to refer to our definition on line four.
There is a reason for this **peculiar syntax**.
Since we're calling this function outside of the module in which it belongs we have to use a fully qualified name:
this is why we prefix it with the module name `FizzBuzz`.
In Elixir functions are distinguish not by their type (signature) but by the module they reside in, their name, and the number of arguments they take (their arity).
This is often abbreviated to MFA (for module, function, and arity) which is ultimately inherited from Prolog through Erlang.

- Prefix a function name, fully qualified or otherwise, with an ampersand `&` to tell Elixir that this "symbol" refers to a function.

- A common higher-order function is `Enum.map/2`.
  It consumes a collection and a function.
  It produces another collection by applying the function to each element in the input collection. Here is an example:

  ```Elixir
  defmodule Mu do
    def square(x) do
      x * x
    end
  end

  Enum.map([0,1,2], &Mu.square/1)
  ```

  Which will give us `[0,1,4]`

- You can find documentation for `Enum.intersperse/2` here: https://hexdocs.pm/elixir/Enum.html#intersperse/2

- The call to `IO.puts/1` will print the collection we've built.

## Testing

Here is one of the test cases we wrote.
It's more like a property-based test than a traditional test:

```Elixir
defmodule FizzBuzzTest do
  use ExUnit.Case

  test "number of `Fizz`es greater than number of `Buzz`es" do
    ## given
    x = Enum.map(1..100, &FizzBuzz.stringify/1)
    ## when
    f = Enum.count(x, fn (s) -> s == "Fizz" end)
    b = Enum.count(x, fn (s) -> s == "Buzz" end)
    ## then
    assert f > b
  end
end
```

Pure functions, i.e. functions that do not read or write IO, lend themselves to testing.
The above demonstrates a well constructed test case with three sections:

1. The _given_ section lists any preconditions

2. The _when_ section does some work

3. The _then_ section lists any postconditions

This is a common practice.

## More Strange Syntax

There are a number of ways to write and represent functions in Elixir.
We've explained two:

- We've seen the `def` and `defp` syntax.

- We've also seen the `&Integer.to_string/1` syntax.

The other is an anonymous function literal.
Just like strings have a literal representation in the source for other languages (e.g. ``"I'm a string!"``), in so-called functional programming languages, functions have a literal representation too.
We can put them in a variable to use them or pass them in directly.
They are a value in their own right:

```Elixir
fizz? = fn (s) ->
  s == "Fizz"
end

buzz? = fn (s) -> s == "Buzz" end

## when
f = Enum.count(x, fizz?)
b = Enum.count(x, buzz?)
```

The question-mark `?`, `!`, and other characters are allowed in function and variable names.
By convention a `?` indicates a predicate (boolean valued function).
