# Functional Swift. Part Two

There are number of concepts in functional programming. We are going to explore a few of them. Their purpose is to make computations more functionally expressible. Those concepts are represented by functional data structures such as `Semigroup` and `Monoid`. From the previous article we figured out that one of the rules of functional paradigm is immutability and more declarative approach in comparison to OOP. Immutability in functional data structures is achieved through creating new versions of the data structures which coexist with the previous version. The parts that need to be changed are copied without affecting the original version of the data structure. That is the basic conceptual model of how functional data structures work. Let’s jump into code to see how it works in practice 🎬

## Semigroup 
`Semigroup` is an algebraic data structure that operates on two elements and returns a new `Semigroup` that has an associative operation. We have two ways to go: since `Semigroup` operates on a set, we can just define a binary operator, or we can define a generic protocol and mix & match POP with Functional Paradigm! I assume that you know what is POP and its principles as well we have a basic experience working with them. Otherwise you can find out more [here](https://docs.swift.org/swift-book/LanguageGuide/Protocols.html).

```swift
infix operator <> : AdditionPrecedence

protocol Semigroup {
    static func <> (lhs: Self, res: Self) -> Self
}
```

We defined an infix operator called `diamond` operator using addition precedence group. You can figure out more about precede groups and custom operators [here](https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations). The operator will serve us as a foundation since we express `Semigroup` through an operation and using protocol extension we are going to define how exactly the operator is going to work on a particular data type. In order to do that, let’s define our first extension for `Int` type:

```swift
extension Int: Semigroup {
    static func <> (lhs: Int, rhs: Int) -> Int {
        return lhs + rhs
    }
}
```

So far looks pretty straightforward and simple. Let’s see how the operator works:

```swift
1 <> 2 <> 3
// Outputs: 6
```

Let’s go a bit further and define several extension for `Semigroup` for `String`, `Bool` and  `Array` types:

```swift
extension String: Semigroup {
    static func <> (lhs: String, rhs: String) -> String {
        return lhs + rhs
    }
}

extension Bool: Semigroup {
    static func <> (lhs: Bool, rhs: Bool) -> Bool {
        return lhs && rhs
    }
}

extension Array: Semigroup {
    static func <> (lhs: Array, rhs: Array) -> Array {
        return lhs + rhs
    }
}
```

`String` semigroup can be made of concatenating two strings, `Bool`semigroup is created using  logical `and` operator and`Array` or array of elements is a concatenation of two arrays. The usage of the operator is still pretty straightforward: 

```swift
“Hello” <> “ ” <> “ World” <> “!”
// Hello World!

[1,2,3] <> [4,5,6,7,8] 
// [1,2,3,4,5,6,7,8,]

true <> false
// false
```

So far we didn’t get much actually! However we have established one important thing, which is a general principle of composition using protocol. This leads us to a pattern, and each pattern has some purposes.  For example we can now shorten the implementation of `reduce` function using `Semigroups`. In order to do that we need to implement new extension for `Array` type. The extension will introduce new instance method for concatenating the supported types:

```swift
extension Array where Array.Element: Semigroup {

    func concat(using initial: Array.Element) -> Array.Element {
        return self.reduce(initial, <>)
    }
}
```

With this extension `Semigroup` extensions start to bring some swifty tools. Let’s take a look how this helps us: 

```swift
[1,3,4,5,6,7].concat(using: 0) // 26
[true, false, true, true].concat(using: true) // false
["Hello", " ", "new ", "world", "!"].concat(using: "") // Hello new world!
```

What we have got here, is simplification of computations: since `Semigroup` already knows about the accumulation, we got rid of one part of `reduce` function, which is the part of accumulation. `Semigroup` naturally knowns about this operation. 

The next part will introduce new functional data structure that even greatly simplifies computations!

## Monoid
`Monoid` is very similar to `Semigroup`. It introduces one additional concept which is the `identity`. Speaking from the technical perspective `identity` is the initial state of the `Monoid`. 

```swift
protocol Monoid: Semigroup {
    static var identity: Self { get }
}

extension Int: Monoid {
    static let identity = 0
}

extension String: Monoid {
    static let identity = ""
}

extension Bool: Monoid {
    static let identity = true
}

extension Array: Monoid {
    static var identity: Array {
        return []
    }
}
```

We introduced new protocol called `Monoid` that has a single read-only property called `identity` of type `Self`. The next is we added conformance of the protocol for the supported types such as `Int`, `String`, `Bool` and `Array`.  

The only thing that is left is to implement `concat` method for `Array` protocol where array element type is `Monoid`:

```swift
extension Array where Array.Element: Monoid {

    func concat() -> Array.Element {
        return self.reduce(Array.Element.identity, <>)
    }
}

[[1,3,4,5,6,7], [8,9,10]].concat() // [1,2,3,4,5,6,7,8,9,10]
[true, false, true, true].concat() // false
["Hello", " ", "new ", "world", "!"].concat() // Hello new world!
```

The does pretty much the same except one addition: we no longer need to use the initial value since each type that supports `Monoid` has the `identity` property. 


## Morphism 
In functional programming `Morphism` is simply a transformation function. There are two types of morphisms: `Endomorphism` and `Isomorphism`. `Endomorphism` is represented as a function where the input type is the same as the output. `Isomorphism` is also represented as a pair of functions (of course, it is Functional Paradigm - everything is a function 😄) that transforms between two types of objects that is structural in nature and no data is lost:

```swift
struct Endomorphism<E> {

    // MARK: - Properties

    let handle: (E) -> E
}

extension Endomorphism: Semigroup, Monoid {

    // MARK: - Conformance to Monoid Protocol

    static var identity: Endomorphism<E> {
        return Endomorphism { $0 }
    }

    // MARK: - Conformance to Semigroup protocol

    static func <> (lhs: Endomorphism, rhs: Endomorphism) -> Endomorphism {
        return Endomorphism { rhs.handle(lhs.handle($0)) }
    }
}
```

`Endomorphism` is represented as `struct` with a single immutable property called `handle`. `Handle` is a closure that accepts a generic parameter `E` and returns it. Then we add conformances for `Monoid` protocol. Since `Monoid` is a subtype of `Semigroup` we have to define `diamond` operator as a static function and define the `identity` property in order to define the initial state of `Endomorphism`.  The diamond operator is simply creates new `Endomorphism` by taking the right hand side `Endomorphism`, calling the `handle` closure with the input parameter of the value that was returned by the left hand side `Endomorphism` function execution. 🤯 I know, it seems like one of those [Christopher Nolan movies](https://www.imdb.com/title/tt1375666/) 😄.

By using this new endomorphic type we can actually do pretty neat things, like the following:

```swift
let decrement = Endomorphism<Int> { $0 - 1 }
let square = Endomorphism<Int> { $0 * $0 }
let increment = Endomorphism<Int> { $0 + 1 }

[decrement, square, increment].concat().handle(3)

// Output:
// 5

// initial value 	-> 3
// decrement 		-> 3 - 1 = 2
// square 		-> 2 * 2 = 4
// increment 		-> 4 + 1 = 5
```

Let’s break it down line by line. We started from defining new properties of type `Endomorphism` of type `Int`.  I hope that each of the structs are pretty self-explanatory. Basically we wrapped logic into functions that are composable and can be handled separately. We then created an array of operations, called the `concat` function (which was defined as an extension for `Array` type) and called `handle` closure with the initial parameter. 
Practical example explained how the `diamond` operator works: sometimes it’s better to understand how the system works from the outside, instead of trying to decompose every element and try to understand from the bottom up. By changing the order and type of the chained functions we can understand the concept behind the operator for `Endomorphism` - it simply takes the argument from `handle` function and passes it to the first endomorphic function and so on for each of the functions, until there is no function to execute. 

Let’s take a look at the last example, where we use `String` type to create composable, endomorphic transformations:

```swift
let prefix = Endomorphism<String> { $0.appending("appLE") }
let space = Endomorphism<String> { $0.appending(" ") }
let postfix = Endomorphism<String> { $0.appending("THE") }
let lowercased = Endomorphism<String> { $0.lowercased() }
let capitalize = Endomorphism<String> { $0.capitalized }

[space, postfix, space, lowercased, prefix, capitalize].concat().handle("cracking")

// Output: 
// Cracking The Apple
```

We started from declaring operations on top of type `String`. Then we created an array of endomorphic functions and we called `concat` function in order to assemble the functions. Then we chained the result with `handle` function that accepts a `String` parameter, which is applied first and then all the operators form the array. The first part that we have is `cracking` string value. Then we add `space` and `postfix` - as a result we have `cracking THE` string. Then we add another `space`, `lowercased` and finally applied `capitalize` endomorphic operation. As a result we transformed `cracking THE` to `Cracking The Apple` string. 


This is actually a very powerful mechanism that allows to combine and compose differnet endomorphic operators and apply them for various transformations. We can chain functions using very sophisticated operators in order to make our code mode declarative, by removing `how` using various small functional building blocks. When we start creating those blocks, it may seem like we are not going much from it, but applying functional design patterns and concepts, as a result, gives us very great, easy to use tools.  


## Conclusion 
Functional programming offers so many great possibilities. Such a declarative approach hides a lot unnecessary details and allows the developers to focus on `what` instead of `how`. The examples demonstrated that even simple functional building blocks, stacked on top of each other allow to bring more composable, easy to extend high-order functions. We saw how `Functional Paradigm` was combined with `POP` and with `OOP`. Each of the paradigms has some domain applications, none of them are bad. They all good in solving different groups of tasks. Modern software development is very diverse, complicated and I believe that applying the listed paradigms where appropriate is the way to go.
