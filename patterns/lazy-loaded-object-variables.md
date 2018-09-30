# Lazy Loaded Object Variables

As more and more object-oriented OpenEdge code is written, more and more object variables and properties are used. There are, however, a few pitfalls associated with them. 

OpenEdge does not allow the `initial` statement to be used with object variables, so the OpenEdge programmer now has a question of where to intialize such variables.

1. In the constructor
2. Right before they are used
3. In the property's `get` method

For a lot of use cases, intializing all of your object variables and properties in the classes constructor is workable, but it makes it harder to add additional constructors. You end up in a situation where your other constructors have to call the default constructor or you have to duplicate the initialization code.

If you initialize them right before they are used, you run into a similar situation. What if you need to add another method that uses the object? Can you guarantee that they are run in the right order?

This pattern focuses on the third method with a small addition.



```openedge
define public property MyObj as MyClass no-undo
    get:
        if not valid-object(MyObj) then
            MyObj = new MyClass().
        return MyObj.
    end class.
    set.
```

This patterns offers us a few advantages-

## The initialization has a single entry point.

This offers us some flexibility. If, later on, `MyClass` needed to use a [factory method][] instead of `new` or needed additional method calls, we would only need to change this one place.

[factory method]: https://en.wikipedia.org/wiki/Factory_method_pattern

## Allows for dependency injection and better unit testing

Because we are not initializing the variable in the constructor and only initializing it if it doesn't exist, there is a window of opportunity for a test framework to inject a [stub][] object via [property injection][], thereby allowing easier unit testing.

[stub]: https://en.wikipedia.org/wiki/Method_stub
[property injection]: https://en.wikipedia.org/wiki/Dependency_injection#Setter_injection

## Lower Resource Usage

By delaying initialization of the variable until it is actually used, we are not using up system resources unnecessarily.
