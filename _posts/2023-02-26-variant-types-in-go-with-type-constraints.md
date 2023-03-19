---
layout: post
title:  "Variant Types in Go with Type Constraints"
date:   2023-02-26 14:02:12 +0100
categories: posts
---

**Variant types,** also known as **tagged unions** or **sum types** are a feature of many programming languages both old (ALGOL 68) and new (Rust). A variable of a variant type can contain a value from a fixed set of types. A **tag** indicates which of the different types is in use.

As an example, assume we want to represent constants that are either a string or an integer. In Haskell, this is easy ([Haskell Playground](https://play-haskell.tomsmeding.com/saved/3VvqDQUb)).
``` haskell
data Constant =
    IntConstant Int 
  | StringConstant String
```
A value of type `Constant` is either an `IntConstant` or a `StringConstant`. Here, `IntConstant` and `StringConstant` are the tags that allow us to extract their respective values.
``` haskell
instance Show Constant where
   show (IntConstant i)    = "int(" ++ show i ++ ")"
   show (StringConstant s) = "string(" ++ show s ++ ")"
```

Go is a language that does not natively support variant types. In this post, we explore different techniques of implementing variant types in Go.


## Technique 1: Unified struct with explicit tag ([Go Playground](https://go.dev/play/p/bwvLpmFc9Bu))

One way to implement a variant type in Go is to create a unified struct that contains fields for all possible types of values with an explicit tag indicating which value is in use.
``` go
type Tag int

const (
    StringTag Tag = iota
    IntTag
)

type Constant struct {
    StringValue string
    IntValue    int
    Tag         Tag
}
```

We have to ensure that the value we set and the tag is in sync. For this purpose, we create constructor functions for each type.
``` go
func NewIntConstant(i int) Constant {
    return Constant{IntValue: i, Tag: IntTag}
}

func NewStringConstant(s string) Constant {
    return Constant{StringValue: s, Tag: StringTag}
}
```

Similar to the Haskell example, we can use the tag to match on the value. However, as Go does not check exhaustiveness, we have to add a default case.
``` go
func (c Constant) String() string {
    switch c.Tag {
    case IntTag:
        return fmt.Sprintf("int(%d)", c.IntValue)
    case StringTag:
        return fmt.Sprintf("string(\"%s\")", c.StringValue)
    default:
        panic(fmt.Errorf("invalid tag: %d", c.Tag))
    }
}
```

We may also want to operate on slices of constants. For example, summing up all integer values.
``` go
func sumIntConstants(consts []Constant) int {
    var sum int

    for _, c := range consts {
        if c.Tag == IntTag {
            sum += c.IntValue
        }
    }

    return sum
}
```

### Advantages / Disadvantages

This approach is easy to extend to more types. But this requires a bit of boilerplate code and is also inefficient memory-wise if we have many types. Another disadvantage is that we have to ensure that the value and tag are always in sync. For this, we can encapsulate the internals by not exporting the fields of `Constant` and creating corresponding setter and getter functions.


## Technique 2: Struct with interface value ([Go Playground](https://go.dev/play/p/wARz6CMYJtd))

Another possibility is to create an interface that is implemented by the types of the variant.
``` go
type ConstantVariant interface {
    isConstantVariant()
}

type IntValue int
type StringValue string

func (i IntValue) isConstantVariant()    {}
func (i StringValue) isConstantVariant() {}

type Constant struct {
    Value ConstantVariant
}
```

As before, we create constructor functions for each type.
``` go
func NewIntConstant(i int) Constant {
    return Constant{Value: IntValue(i)}
}

func NewStringConstant(s string) Constant {
    return Constant{Value: StringValue(s)}
}
```

Now, instead of switching on the explicit tag, we do a type switch on the interface value.
``` go
func (c Constant) String() string {
    switch v := c.Value.(type) {
    case IntValue:
        return fmt.Sprintf("int(%d)", v)
    case StringValue:
        return fmt.Sprintf("string(\"%s\")", v)
    default:
        panic(fmt.Errorf("invalid type: %d", c.Value))
    }
}
```

Summing all integers in a slice of constants is similar to the first technique.
``` go
func sumIntConstants(consts []Constant) int {
    var sum int

    for _, c := range consts {
        v, ok := c.Value.(IntValue)
        if ok {
            sum += int(v)
        }
    }

    return sum
}
```

### Advantages / Disadvantages

The most striking advantage is that we do not need an explicit tag. We use the type as an implicit tag.
A minor disadvantage occurs when we compare constants since they contain interface values. If we have constants containing 'int8(1)' and int16(1)", '==' would return false. An interface value consists of two pieces: (1) a type and (2) a value of that type. Regarding the comparison of two interface values, the [Go Specification](https://go.dev/ref/spec#Type_identity) says:
> Two interface values are equal if they have [identical](https://go.dev/ref/spec#Type_identity) dynamic types and equal dynamic values or if both have value `nil`.  


## Technique 3: Top-level type constraint ([Go Playground](https://go.dev/play/p/t_Fn47kEb61))

A third technique makes use of generics and type constraints that have been introduced in [Go 1.18](https://tip.golang.org/doc/go1.18).
Type constraints in Go look very similar to untagged unions in [TypeScript](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types) and [PHP](https://php.watch/versions/8.0/union-types).
``` go
type ConstantConstraint interface {
    int | string
}
```

Defining the `Constant` type should then be straightforward shouldn't?
``` go
type Constant struct {
    Value ConstantConstraint
}
```
Let us try this on the [Go Playground](https://go.dev/play/p/kKl2LSins7g).
``` go
./prog.go:8:8: cannot use type ConstantConstraint outside a type constraint: interface contains type constraints
```
Oops, Go does not allow us to use an interface that contains a type constraint as a regular type for `Value`.
To use our type constraint, we have to add a type parameter to `Constant`.
``` go
type Constant[T ConstantConstraint] struct {
    Value T
}
```

Now, Go is happy 😄 ([Go Playground](https://go.dev/play/p/wbKWttHXVMF)).

We could create constructor functions. However, the syntax for creating new constants is already very concise.
``` go
Constant[int]{1337}
Constant[string]{"Hello, world!"}
```

We have to instantiate the type parameter `T` in the definition of `Constant` with the concrete type. Go checks that the type satisfies the type constraint in `ConstantConstraint`. If the type constraint is violated, for example, as in `Constant[bool]{true}`, we get a compile-time error ([Go Playground](https://go.dev/play/p/sNo-iHlKGS-)).
```
./prog.go:14:23: bool does not satisfy ConstantConstraint (bool missing in int | string)
```

We now also have to add a type parameter `T` for `Constant` to the member function `String()`.
``` go
func (c Constant[T]) String() string {
    switch v := c.Value.(type) {
    case int:
        return fmt.Sprintf("int(%d)", v)
    case string:
        return fmt.Sprintf("string(\"%s\")", v)
    default:
        panic(fmt.Errorf("invalid type: %d", c.Value))
    }
}
```
Unfortunately, Go does not let us use a type switch on a constraint type ([Go Playground](https://go.dev/play/p/FG8_cVLkdJs)).
```
./prog.go:14:14: cannot use type switch on type parameter value c.Value (variable of type T constrained by ConstantConstraint)
```
However, with a trick ([Source](https://appliedgo.com/blog/a-tip-and-a-trick-when-working-with-generics)), we can overcome this restriction. Instead of switching on the type of `c.Value`, we first convert it to an empty interface on which we can then do a type switch.
``` go
func (c Constant[T]) String() string {
    switch v := any(c.Value).(type) {
    case int:
        return fmt.Sprintf("int(%d)", v)
    case string:
        return fmt.Sprintf("string(\"%s\")", v)
    default:
        panic(fmt.Errorf("invalid type: %d", c.Value))
    }
}
```

When we want to sum over all integers in a slice of constants, we face a problem. Since there is no unified type of constants (`Constant[int]` and `Constant[string]` are two completely different types), the type of the slice has to be `[]any`. We therefore have to do a type assertion from `any` to `Constant[int]`.
``` go
func sumIntConstants(consts []any) int {
    var sum int

    for _, c := range consts {
        v, ok := c.(Constant[int])
        if ok {
            sum += v.Value
        }
    }

    return sum
}
```

### Advantages / Disadvantages

A major advantage of this approach is the concise syntax and the compile-time type checking. It is very easy to extend by adding a type to `ConstantConstraint` and does not require any boilerplate code.
A disadvantage is, as we have seen above, that we have to use `any` when we want to create a heterogenous slice since this is the only common type for all constants.

## Technique 4: Wrapped type constraint ([Go Playground](https://go.dev/play/p/PJ_RoIyoN84))

We can combine techniques 2 and 3 by wrapping a type constraint by `ConstantConstraint` in another struct.
``` go
type ConstantConstraint interface {
    int | string
}

type ConstantVariant interface {
    isVariant()
}

type Constant struct {
    Value ConstantVariant
}

type Variant[T ConstantConstraint] struct {
    unwrap T
}

func (v Variant[T]) isVariant() {}
```

Here, we create a generic constructor function whose type parameter is constraint by `ConstantConstraint`.
``` go
func NewConstant[T ConstantConstraint](v T) Constant {
    return Constant{Variant[T]{v}}
}
```

For `String()`, we type switch on `c.Value` and compare it against instances of `Variant[T]`.
``` go
func (c Constant) String() string {
    switch v := c.Value.(type) {
    case Variant[int]:
        return fmt.Sprintf("int(%d)", v.unwrap)
    case Variant[string]:
        return fmt.Sprintf("string(\"%s\")", v.unwrap)
    default:
        panic(fmt.Errorf("invalid type: %d", c.Value))
    }
}
```

Summing over all integers in a slice of constants is very similar to the second technique.
``` go
func sumIntConstants(consts []Constant) int {
    var sum int

    for _, c := range consts {
        v, ok := c.Value.(Variant[int])
        if ok {
            sum += v.unwrap
        }
    }

    return sum
}
```

### Advantages / Disadvantages

This technique combines the advantages of technique 2 of not requiring an explicit tag with the ones from technique 3, a concise syntax and compile-time type checking. Additionally, we overcome the disadvantage of technique 3 of using `any` since we `Constant` is now the unified type for all constants.

## Conclusion

While Go does not natively support variant types, it is not too difficult to implement them with good ergonomics. In this post, we explored four possible techniques each with different advantages and disadvantages. What technique to use depends on the context and your requirements. I am sure there are ways I did not think of.

For my use case, encoding symbolic expressions, I went with the fourth technique due to its conciseness and flexibility.
