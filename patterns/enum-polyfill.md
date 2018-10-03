# Enum Polyfill

OpenEdge has included support for the enumerated types (Enums) since 11.4. Enums are very useful in that you can provide semantic meaning to special values, get compile-time validation, and benefits like design autocomplete. The problem comes in if you need to use an earlier version of OpenEdge. Below is a pattern to create an (almost) API compatible enum in versions before 11.4.

## Base Class

You'll want to create a base class for all of your enums to inherit from. This will handle the basic methods and properties.

```OpenEdge
class EnumMember abstract:

    define public property Name as character no-undo
        get.
        protected set.
    
    define public property Value as integer no-undo
        get.
        protected set.

    constructor protected EnumMember(enumName as character, enumValue as integer):
        this-object:Name = enumName.
        this-object:Value = enumValue.
    end constructor.

    method public override ToString():
        return this-object:Name.
    end method.

    method public Equals(compareEnum as EnumMember):
        return this-object:Value = compareEnum:Value.
    end method.

end class.
```

Unfortunately, OpenEdge does not allow operator overloading, so we cannot use comparison operators like the builtin enums. Instead, we do like Java and add methods like `Equals` to compare enums. 

The real magic happens in the subclasses, however.

## Implementation

In the subclass, we create a static property for each value that is needed. This static value uses the singleton pattern to reduce overhead. Theoretically, this should allow us to use `=` for equality since there will only be one object with the right memory location. 

```
class Direction
    inherits EnumMember:

    constructor protected Direction(enumName as character, enumValue as integer):
        super(enumName, enumValue).
    end constructor.

    define public static property North as Direction no-undo
        get:
            if not valid-object(Direction:North) then
                Direction:North = new Direction("North":U, 1).
            return Direction:North.
        end get.
        protected set.

    define public static property South as Direction no-undo
        get:
            if not valid-object(Direction:South) then
                Direction:South = new Direction("South", 2).
            return Direction:South.
        end get.
        protected set.

    define public static property East as Direction no-undo
        get:
            if not valid-object(Direction:East) then
                Direction:East = new Direction("East", 3).
            return Direction:East.
        end get.
        protected set.

    define public static property West as Direction no-undo
        get:
            if not valid-object(Direction:West) then
                Direction:West = new Direction("West", 4).
            return Direction:West.
        end get.
        protected set.

    method public static Direction GetEnum(enumName as character):
        case enumName:
            when North:Name then
                return North.
            when South:Name then
                return South.
            when East:Name then
                return East.
            when West:Name then
                return West.
            otherwise
                undo, throw new Progress.Lang.AppError("Invalid Enum Name").
        end case.
    end method.

    method public static Direction GetEnum(enumValue as integer):
        case enumValue:
            when North:Value then
                return North.
            when South:Value then
                return South.
            when East:Value then
                return East.
            when West:Value then
                return West.
            otherwise
                undo, throw new Progress.Lang.AppError("Invalid Enum Value").
        end case.

    end method.

end class.
```

The static properties allow us to use it in a similar way to a builtin enum-

```
define variable myDirection as Direction no-undo.

myDirection = Direction:South.
```

The static `GetEnum()` methods allow us to de-serialize from a string or integer depending on our needs. Unforuantely, there doesn't seem to be a way to move these methods into the base class.

The downsides of this method-

- It's verbose
- Enum objects live for the lifetime of the session
- It does not work for flag enums

I suspect that all of these are solveable. You can use macros in your IDE to generate most of the code. You could also make a [JET template][jet] to generate the whole class. You could add a dispose or delete method to delete the object references. You could start with a different base class for flag enums.

Obviously, if you can, you should use the built-in enums, though.

[jet]: https://www.eclipse.org/articles/Article-JET/jet_tutorial1.html