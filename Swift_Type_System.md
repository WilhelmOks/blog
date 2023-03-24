# Swift's Type System

This is a summary of my thoughts and findings about Swift's type system and how it differs from other programming languages.<br>
It assumes a basic understanding and familiarity with Swift and explains some interesting details that you might not be aware of.

## Passing and returning data

Functions or methods can have parameters. They can have many parameters, just one parameter or no parameters at all.<br>
This is nothing unusual in the world of programming and anyone can think of several use cases for an arbitrary number of parameters.

When we look at what functions can return though, for most programming languages the realization is: Just one value or nothing (typically indicated by "void").<br>
The question is: Wouldn't it be great to be able to ruturn many values just like we can pass many values as parameters?

Some languages (C#) have added support for tuples at a later point in their development to allow to return more than one value at once.<br>
And this is great. But since they are just additions, they don't feel very natural to the language and are technically just anonymous structures which hold multiple values wrapped in a single value.<br>
Or, in the case of JavaScript, arrays are used as "a poor man's" tuples.

## Let's take a look at tuples in Swift

Swift's tuples are a fundamental part of Swift's design from the very beginning.

A tuple can hold 
* many values: `(value1, value2, value3)`
* one value: `(value)`
* or no values: `()`

In any case, it just looks like a list of parameters. And tuple values can have labels just like function parameters, too.<br>
Technically, function parameters are not actually tuples, but conceptually we can think of passing parameters and returning values as being the same thing and having the same syntax.

Everything is a tuple.

A normal value is actually a tuple with one single value. In that case, we are allowed to omit the brackets for better readability.
```
let valueA = (5)
let valueB = 5
// valueA is the same as valueB
```

A tuple with no values (empty tuple) is the same as `Void`.
In Swift, we normally omit `Void` when we don't return anything from a function.<br>
Implicitly, we just return 0 values, which is represented by an empty tuple `()` or `Void`.

Even though an empty tuple represents 0 values, it can still be seen as a value itself.<br>
We can store that value in a variable and pass it around like any other value:
```
let noValues1 = ()
let noValues2 = Void()
```
This is a somewhat special value though, because it is the only possible value that an empty tuple can have.

You might wonder how this is useful.<br>
One interesting use case is Swift's result builders.

To make it more clear, let's take a look at SwiftUI:<br>
In SwiftUI, there is a special result builder `ViewBuilder`, which builds a `View` from a list of expressions which also evaluate to `View`, respectively.<br>
A result builder can consume an arbitrary amount of expressions, including 0 expressions.<br>
So, a `View` actually represents many views, one view, or no views. And there is a special view that represents 0 views: `EmptyView`.<br>
When we are building a `View` based on a set of conditions and the compiler wants us to specify a `View` for a case where we don't want to have any view, we just provide an `EmptyView()`.

Similarly, when we have result builders that work with any kind of values, not just those conforming to `View`, we can use `Void` to represent 0 values, just like we use `EmptyView()` to represent 0 views.<br>
For a concrete example, you can take a look at my implementation of ArrayBuilder: https://github.com/WilhelmOks/ArrayBuilder

So, even when we return 0 values, we still return something.<br>
But what if we actually don't want to return anything, not even `Void`?<br>
There is a type for that and it's called `Never`.

## What is Never and what is it used for?
`Never` is a type that you can never have an instance of.<br>
So, it's impossible to pass around or return a value of `Never`.

One example to illustrate the use of `Never` is the global function `fatalError()`.<br>
When called, it just terminates the program with a specified message.<br>
This function has `Never` as its return type and it means that it never returns.<br>
Because of the special type `Never`, the compiler will give us a warning when we call `fatalError` with some other code statements after that.<br>
Since `fatalError` never returns, the code after it will never be executed.<br>
The compiler will tell us, that assuming that the code after `fatalError()` will be executed, is a mistake. Nice!

For another example where `Never` is useful, let's take a look at the `Result` type that is part of Swift's standard library.<br>
`Result` has two generic parameters, `Success` and `Failure`.<br>
`enum Result<Success, Failure> where Failure : Error`<br>
We can specify any combination of `Success` and `Failure` types, which represent a result for some operation that may succeed or fail.<br>
But what if we want to specify a `Result` that never fails?

For example, using the `Combine` framework to publish the stream of values from a textfield, we would have `String` as the `Result`'s `Success` type. <br>
But since it will never produce any errors or fail, what should be the `Failure` type?<br>
It wouldn't be appropriate to use `Void` as the `Failure` type, as this would indicate that it can fail. And the only possible way of failure would be represented by the only possible value that `Void` can have: `Void()`.<br>
And the compiler would force us to handle this failure case by explicitly checking if the `Result` is `.success()` or `.failure()`.

Using `Never` as the `Failure` type solves the problem: <br>
It communicates the intent that the `Result` will never be a failure.<br>
And it also enables the compiler to completely eliminate the `.failure()` case. We are no longer forced to handle it in code and it becomes more readable.

In general, `Never` is a great tool to use the type system to make code more correct and clear, without any compromises.

## Conclusion
Swift's type system looks very similar to those of other C-like languages on the surface. <br>
But looking closer, it has a very logical and consistent design, which enables us to model data and logic in the most safe and flexible way.
