# Chapter 3 - Functions
Functions are the first line of organization in any program. They are "self contained" modules of code that accomplish task. Writing them well makes your code easier to read and maintain.

## Some (recommended) rules for writing good functions
### Small!
The first rule of functions is that they should be small. *The second rule of functions is that they should be smaller than that.*
#### Block and Indenting
Block of code should be in the same indentation. The indent level of a function should not be greater than two, more than that then we can't really follow all the possibilities.
### Do One Thing
> FUNCTIONS SHOULD DO ONE THING. THEY SHOULD DO IT WELL. THEY SHOULD DO IT ONLY.
#### Problematic Sign: Sections within Functions
If a function is divided into sections such as declarations, initializations and process, this is an obvious symptom of doing more than one thing. Functions that do one thing cannot be reasonably divided into sections.
### One Level of Abstraction per Function
In order to make sure our functions are doing “one thing”, we need to make sure that the statements within our function are all at the same level of abstraction. Mixing levels of abstraction within a function is always confusing. Readers may not be able to tell whether a particular expression is an essential concept or a detail. Worse, like broken windows, once details are mixed with essential concepts, more and more details tend to accrete within the function.
#### Reading Code from Top to Bottom: *The Stepdown Rule*
We want the code to read like a top-down narrative. We want every function to be followed by those at the next level of abstraction so that we can read the program, descending one level of abstraction at a time as we read down the list of functions.
### Switch statements
It is hard to make small `switch` statement. By their nature, `switch` statements always do N things. Unfortunately we can't always avoid `switch` statements, but we *can* make sure that each `switch` statement is buried in a low-level class and is never repeated.
### Use Descriptive Names
Don’t be afraid to make a name long. A long descriptive name is better than a short enigmatic name. A long descriptive name is better than a long descriptive comment. Modern IDEs like Eclipse or IntelliJ make it trivial to change names, so we should spend time choosing good names.
### Function Arguments
The number of arguments for a function should not be more than two. Three arguments should be avoided where possible. More than three requires very special justification.

Output arguments are harder to understand than input arguments. When we read a function, we are used to the idea of information going in to the function through arguments and out through the return value. We don’t usually expect information to be going out through the arguments.
#### Flag Arguments
>Flag arguments are ugly. Passing a boolean into a function is a truly terrible practice. It immediately complicates the signature of the method, loudly proclaiming that this function does more than one thing. It does one thing if the flag is true and another if the flag is false!

We should split into more functions instead of passing boolean as argument
#### Problems with functions having more than one argument
What is the order of the argument? How many time did we write our test cases with `assertEquals(actual, expected)` instead of `assertEquals(expected, actual)`?
#### Argument Objects
When a function seems to need more than two or three arguments, it is likely that some of those arguments ought to be wrapped into a class of their own.
#### Argument Lists
Sometimes we want to pass a variable number of arguments into a function. If the variable arguments are all treated identically, they can be passed as one argument of type `List`
#### Verbs and Keywords
Function name should be a verb. We can use keywords to encode the names of the arguments into the function name. `assertEquals` might be better written as `assertExpectedEqualsActual(expected, actual)`.
### Have No Side Effects
Side effects are lies. Your function promises to do one thing, but it also does other *hidden* things.
#### Output Arguments
In the days before object oriented programming it was sometimes necessary to have output arguments to modify input. However, much of the need for output arguments disappears in OO languages because **this** is intended to act as an output argument. In general output arguments should be avoided. If your function must change the state of something, have it change the state of its owning object.
### Command Query Separation
Functions should either do something or answer something, but not both. E.g. of a function that does both:
`public boolean set(String attribute, String value);`
### Prefer Exceptions to Returning Error Codes
Returning error codes from command functions is a subtle violation of command query separation. It promotes commands being used as expressions in the predicates of if statements.
```
if (deletePage(page) == E_OK)
```
It could lead to deeply nested structures. When you return an error code, you create the problem that the caller must deal with the error immediately. It is preferred to use exception and try/catch block instead.
#### Extract Try/Catch Blocks
These blocks confuse the structure of the code and mix error processing with normal processing. So it is better to extract the bodies of the try and catch blocks out into functions of their own.
#### Error Handling Is One Thing
Functions should do one thing. Error handing is one thing. Thus, a function that handles errors should do nothing else. This implies that if the keyword try exists in a function, it should be the very first word in the function and that there should be nothing after the catch/finally blocks.
#### The Error.java Dependency Magnet
Returning error codes usually implies that there is some class or enum in which all the error codes are defined. Classes like this are a dependency magnet; many other classes must import and use them.
### Don't Repeat Yourself
If codes are duplicate, there is high chance that it will be duplicated even more. Use a function instead.
### How To Write Good Functions?
Writing software is like any other kind of writing. When you write a paper or an article, you get your thoughts down first, then you massage it until it reads well. The first draft might be clumsy and disorganized, so you wordsmith it and restructure it and refine it until it reads the way you want it to read.

Writing functions is similar, we write long and complicated functions first. They have lots of indenting and nested loops. They have long argument lists. The names are arbitrary, and there is duplicated code. But we should already have tests to cover the output of that code.

Then we can refine that code, splitting out functions, changing names, eliminating duplication. We shrink the methods and reorder them. We can even break out whole classes, while we keep the tests passing.