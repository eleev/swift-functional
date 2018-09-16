# Functional Swift. Part One

This is the first article about functional programming paradigm in the context of Swift programming language. The first part of 
the series is going to be about the overview and explanation of the built in functional capabilities of the language. We will talk 
about imperative vs declarative programming approaches and see how to apply all the knowledge in practice. The article is for software engineers who already know the basics of programming and software development. Without further ado -
let's get started! 🎬

## Functional Paradigm 
There are a number of different approaches in building software. Hopefully you are already familiar with `Object Oriented Paradigm` and probably `Protocol Oriented Paradigm`. To put it simply `Functional Paradigm` is an another way to structure and build programs. As the name suggests, *Functional Paradigm* is built around functions and a couple of additional rules, such as:

* Immutability of data 
* Programming is done using declarations rather than statements 
* Eliminated side effects 
* Recursion pattern is used over imperative iterations (such as `for`, `for-in` etc.)
* Closures and high order functions 
* Functional composition 

The aforementioned characteristics are essential for *Functional Paradigm* and Swift offers all of them out of the box. Using Swift we are able to easily change the state of a property from mutable to constant by using the `var` and `let` keywords. Declarative programming is achieved by hiding the underlying imperative statements using higher level abstractions. Elimination of sides effects is achieved through strongly immutable objects (strongly immutable object is an object that is constructed only once and never changes). As well as Swift supports functions as first class citizens, which means that we can:

* Compose functions and capture the surrounding context 
* Nest functions 
* Pass functions around as parameters 
* Create functions that accept and return n-number of other functions 
* We can even create our own custom operators that perform operations under functions (if you think about it, it’s actually a very powerful mechanism)

Most of the mechanisms will be shown in some form during the `Functional Swift` series.

## Practical Perspective

In order to more practically understand how declarative approach looks like vs imperative alternative, let’s program something very simple and take a look at the differences. Let’s take as an example the following task: `To build a function or a series of functions that filter arrays using logical condition`. 

Sure, not an issue at all. We start from the imperative, structural way of building program by declaring function, its signature and implementation. 

```swift
func filterOdd(_ numbers: [Int]) -> [Int] {
    var  output = [Int]()
    
    for number in numbers {
        if number % 2 == 1 {
            output += [number]
        }
    }
    return output
}

func filterEven(_ numbers: [Int]) -> [Int] {
    var  output = [Int]()

    for number in numbers {
        if number % 2 == 0 {
            output += [number]
        }
    }
    return output
}
```

The presented functions filter out a set of values of the input arrays using different logical conditions. The fact that we need to reuse almost the same code over and over in each function is error prone, not efficient and hard to read (in this particular example it's not that hard, but let’s extrapolate and think about complex data base filters and data transformations). We can use the power of Swift’s closures and implement more weakly-coupled function by injecting executable code inside the filter function. Our injectable closure accepts one `Int` parameter and returns `Bool` value which serves as logical filter. 

We can implement the following function using Swift’s closures: 

```swift
func filter(_ numbers: [Int], condition: (Int)->Bool) -> [Int] {
    var output = [Int]()

    for number in numbers {
        if condition(number) {
            output += [number]
        }
    }
    return output
}
```

Which replaces the mentioned functions:

```swift
let numbers = [1,2,3,4,5,6,7,8,9]
let oddNumbers = filter(numbers) { number -> Bool in
    return number % 2 == 0
}

let evenNumbers = filter(numbers) { number -> Bool in
    return number % 2 == 1
}
```

Conceptually not so much have changed. However we were able to eliminate boilerplate, strongly-coupled code. We can use even shorter version by removing the closures' signature and return statement:

```swift
let evenNumbers = filter(numbers) { $0 % 2 == 1 }
```

We also can implement some additional functions that filter out the given array of data:

```swift
// Assume we have an array of grades where 5 is the highest grade 
let highestGrades = filter(numbers: grades, { number -> Bool in
    return number == 5
})
```

In order to make it more Swifty we implement this function as an extension for `Array`:

```swift
extension Array where Array.Element:Comparable {
    func filter(with condition: (Array.Element) -> Bool) -> Array {
            var output = [Array.Element]()

        for element in self where condition(element) {
            output += [element]
        }
        return output
    }
}

let grades = [1,1,2,3,5,5,3,3,5,2,3,4,4,2,3,4]
let highestGrades = grades.filter { $0 == 1 }
```

Let’s break the example down. We took the already familiar code for filter function and wrapped it into an extension for `Array` type. Since we need to perform logical comparison between array elements we need to make sure that the corresponding element type conforms to `Comparable` and `Equatable` protocols. Since `Comparable` protocol already conforms to `Equatable` we may not use protocol composition for `Equatable` and keep the extension declaration as simple as possible. The next part is to replace the concrete type to the generic array element type. And as a final touch we can use Swift’s pattern matching capabilities to make the for-in loop more readable and without nested if statement. 

The provided example shown us how imperative, structured code can be used to create functional, declarative, high order functions that make some operations more convenient to implement, test and reuse. 


## Built In Functional Capabilities 

You may be surprised, but Swift already has similar functionality. Swift has the following high-order functions:

* filter 
* map 
* flatMap
* compactMap
* reduce

The list is not complete, there are many more functions. The thing is that if you understand the concepts behind the listed functions, you will be able to easily extend your knowledge and use the remaining functions, design and implement your own versions.

Let’s start from already familiar function called `filter`. Here is how it is defined in Swift’s documentation for `Array` type:

> Returns an array containing, in order, the elements of the sequence that satisfy the given predicate.

And for `Dictionary` type:

>  Returns a new dictionary containing the key-value pairs of the dictionary that satisfy the given predicate.

Conceptually the description is the same for `Set` type. And that is pretty much what we have already done before. However, our implementation only worked with `Arrays`. Swift provides implementations of this function for all the built in data structures such as `Set`, `Dictionary` and `Array` out of the box. 

```swift
let names = ["John", "Steve", "Alex", "Sandy"]
var result = names.filter { $0.first == "S" }
```

The interface is pretty much the same, nothing has changed in comparison to our implementation. Let’s move on to `map` function and its alternatives such as `flatMap` and `compactMap`. 

Conceptually `map` can be understood as `transform each element of a collection using a closure`. Official documentation defines the functions as follows:

>  Returns an array containing the results of mapping the given closure over the sequence’s elements.

With `map` function we can greatly reduce complexity of code in situations when we need to transform collections of data. `Map` function has the following three forms: 

- `map` - this is the “classic” version of the function. It takes a collection of data and applies some closure that transforms the data. The function does nothing in cases when we have nested collections, nil and optional values. 
- `flatMap` - this version of the fiction has a couple of interesting characteristics. The first one, as the name suggests, is the function that flattens nested collections and applies already familiar `map` function. Basically you can think of this function as a composition of two functions: `flatten` (which is an actual function that merges nested collections into a single one) and `map` function. The second thing to note about this function is that is does unwrap optionals and removes nil values from collections. That is actually very convenient in many cases. 
- `compactMap` function is basically the same as `map` but it returns a non-nil results of calling the given transformation with each element for the given sequence. 

```swift
let dict = ["J": [1,2,3,4,5,8], "A": [6,7,7], "B": [8,9], "H": [5,6,5]]

var filteredDict = dict.flatMap { $0.value.filter{ $0 > 5 }.filter { $0 % 2 == 1 } }

// Prints:
// [9, 7, 7]
```

As the result we will transform a dictionary into an array that contains numbers that are greater than 5 and odd. I intentionally used only shorthand closure names instead of explicitly declaring the signatures for each closure with the `return` statements - since we work only with a single argument inside each of the closures, we can be sure that we will not mess up with the input arguments. 

That was a declarative example. Now let’s take a look at the imperative counterpart that does exactly the same. 

```swift
var filteredDictImperative = [Int]()

for pair in dict {
    let values = pair.value

    for value in values where value > 5 && value % 2 == 1 {
        filteredDictImperative += [value]
    }
}

// Prints:
// [9, 7, 7]
```

That is much harder to understand despite of simplicity of the data transformation! The time that you spend on developing this functional way of thinking will definitely pay off in the future! 😉

We have the last function which is `reduce` function. This function is a bit different. The aforementioned examples transformed sequences into sequences. The `reduce` function transforms sequence into a single value! 

```swift
let names = ["John", "Steve", "Alex", "Sandy"]
let combined = names.reduce(into: "") { (result: inout String, value) in
    result += " " + value
}

// Prints:
// John Steve Alex Sandy
```

## Functional Composition 
That was a lot of material to digest! Brace yourself, we have the final part to crack the topic - which is `functional composition`. Don’t be scared or confused, fictional composition is actually very simple thing to understand. Conceptually it works in this way: you simply pass the output of one function as input to the other one. We already have seen functional composition before: where we used `map` function. Let’s recap and modify the example:

```swift
let dict = ["J": [1,2,3,4,5,8], "A": [6,7,7], "B": [8,9], "H": [5,6,5]]

var filteredDict = dict.flatMap { $0.value.filter{ $0 > 5 }.filter { $0 % 2 == 1 } }

// Prints:
// [9, 7, 7]
```

The part where we compose the output, transformed data of dictionary as an input parameter to `filter` function call, where we assembled all the values into the final array. We could brake it down line by line in the following manner: 

```swift
let flattenOut = dict.flatMap { $0.value.filter{ $0 > 5 } }
// [6, 7, 7, 6, 8, 9, 8]
let filtered = flattenOut.filter { $0 % 2 == 1 }
// [9, 7, 7]
```

However, I do recommend to not overuse functional composition too much, since you may end up in a situation when even the original creator of a code, over time, will not be able to immediately understand what the program does. Break it down into simple, discrete pieces, so each step is transparent and requires as less documentation as possible. 


## Conclusion 
This article concludes the introduction into practical `Functional Programming` in Swift. We talked abut the concepts behind the parading, made a practical comparison between *declarative* and *imperative* approaches. Implemented declarative version of the `filter` function that is used in Swift and get familiar with the built-in functions such as `map`, `flatMap`, `compactMap`, `filter` and `reduce`. 

*Functional Paradigm* has disadvantages as well. Mutable state is not always bad. For instance in game development it is very convenient to create a single component, pass it into different entities and have an ability to change their behavior using a single function, instead of managing each individual value type. Mixing `Functional`, `OOP` and `POP` where appropriate is beneficial, since each of the paradigms has something amazing to offer. 

We could talk about different, real-world use cases in data processing, graphics programming, networking etc. all day long. The article is already long enough. I will try to dive deeper into the subject in the next parts of the series. Thanks for reading and I hope you learnt something new. 
