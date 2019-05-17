# Chapter 14 - Successive Refinement

This chapter is a case study in successive refinement. You will see a module that started well but did not scale. Then you will see how the module was refactored and cleaned.


## Case Introduction

Most of us have had to parse command-line arguments from time to time.Args is very simple to use. You simply construct the Args class with the input argu- ments and a format string, and then query the Args instance for the values of the argu- ments

```java
public static void main(String[] args) { 
    try {
        Args arg = new Args("l,p#,d*", args); 
        boolean logging = arg.getBoolean('l');
        int port = arg.getInt('p');
        String directory = arg.getString('d'); executeApplication(logging, port, directory);
    } catch (ArgsException e) {
        System.out.printf("Argument error: %s\n", e.errorMessage());
    } 
}
```

## Args Implementation

```java
public class IntegerArgumentMarshaler implements ArgumentMarshaler { 
    private int intValue = 0;
    
    public void set(Iterator<String> currentArgument) throws ArgsException { 
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            throw new ArgsException(MISSING_INTEGER);
        } catch (NumberFormatException e) {
            throw new ArgsException(INVALID_INTEGER, parameter); 
        }
    }
    
    public static int getValue(ArgumentMarshaler am) {
        if (am != null && am instanceof IntegerArgumentMarshaler){
            return ((IntegerArgumentMarshaler) am).intValue; 
        }else{
            return 0; 
        }
    }
}
```

## How Did I Do This?
Let me set your mind at rest. 

I did not simply write this program from beginning to end in its current form. More importantly, I am not expecting you to be able to write clean and elegant programs in one pass. 

Programming is a craft more than it is a science. To write clean code, you must first write dirty code and then clean it.

## Args: The Rough Draft
It “works.” And it’s messy.

Notice that the latter mess has only two more argument types than this: String and integer. The addition of just two more argument types had a massively negative impact on the code. 

## So I Stopped
I had at least two more argument types to add, and I could tell that they would make things much worse. 
So I stopped adding features and started refactoring. Having just added the String and integer arguments, I knew that each argument type required new code in three major places. 

- First, each argument type required some way to parse its schema element in order to select the HashMap for that type. 
- Next, each argument type needed to be parsed in the command-line strings and converted to its true type. 
- Finally, each argument type needed a getXXX method so that it could be returned to the caller as its true type.

Many different types, all with similar methods—that sounds like a class to me. And so the ArgumentMarshaler concept was born.

## On Incrementalism
One of the best ways to ruin a program is to make massive changes to its structure in the name of improvement.

To avoid this, I use the discipline of Test-Driven Development (TDD).

## String Arguments
Now let’s look at the whole picture again.

Much of good software design is simply about partitioning—creating appropriate places to put different kinds of code. This separation of concerns makes the code much simpler to understand and maintain.


## Conclusion
**It is not enough for code to work.**

**Keep your code as clean and simple as it can be. Never let the rot get started.**


