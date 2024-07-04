# Revamp of automatic constructors.

* Status: Dicussion
* Discussion: [#1800](https://github.com/nitlang/nit/issues/1800)
* Implementation: todo

## Summary of the previous proposal

The previous proposition comes from #679, more than one year ago.
It introduced *new-style* constructors where the semantic of the `new` is a specific 3-phase construction.

For instance, with the following class `Foo`

~~~nit
class Foo
   var i: Int
end
~~~

the `new` semantic of `x = new Foo(1)` means

~~~nit
x = new Foo.intern # memory allocation and initialization of the default attributes
x.i = 1 # Collected initializers (mostly attribute setters)
x.init # Linearized call to anonymous init to finish the construction
~~~

The benefit of these approach is that the collection of the initializers can be done sanely in multiple inheritance.
The drawback of the approach is that the whole magic is done in the `new` expression and that these automatic constructions, especially the signature of the `new`, are not exposed like other properties in the doc of the class (if exposed at all).

## Proposal of the revamp

This revamp tries to address these issues while keeping the benefits and being compatible with the existing code.
The basic idea is to move the implicit sequence of code of the `new` expression into a genuine method of the class.
Since each class possibly has its own specific sequence of code with its specific signature, these genuine methods will not be redefinition but introduction.
The proposed name for these methods is `auto` and they will behave like named constructors.

So, in the proposal, the `new Foo` is a shorthand for `new Foo.auto`, so a standard 2-phase construction with the named constructor `auto`, like the following:

~~~nit
x = new Foo.intern # memory allocation and init of default attributes
x.auto(1) # call of the named constructor
~~~

The `auto` constructor is implicitly defined in the class, like if it was defined by the user:

~~~nit
# The implicit and free, constructor `auto` 
init auto(i: Int) do
   self.i = i 
   self.init # Linearized call to the anonymous init to finish the construction
end
~~~

Advantages on the revamp:

* the default `new` has a simpler semantic with less hacks.
* the *automatic* construction is exposed as an implicit named constructor `auto`, that behave and is documented like other named constructors.


## Constructions and annotations

The annotations `autoinit`, `noautoinit` and related keep their effects.
The change is that instead of changing the *initializers* that are magically invoked with the `new`, they just change how the implicit `auto` named constructor is set up.
This has the advantage that the effects of these annotations are limited to how the `auto` constructor is defined.

* `noautoinit` on attributes: the setter is not collected in the body of `auto`
* `autoinit` on methods: the method is collected in the body of `auto`
* `autoinit` in the class: the `auto` constructor is made with the setters given in argument in order

The last annotation (`autoinit` on classes) has therefore a simpler meaning since

~~~nit
class Bar
  autoinit x, y, z
  # ...
end
~~~

Is equivalent to

~~~nit
class Bar
   init auto(x: X, y: Y, z: Z) do
      self.x = x
      self.y = y
      self.z = z
      self.init
   end
end
~~~

## Manual named constructors

Most of the time, manual named constructors require that a primitive constructor is called.
In the previous proposal, the magic sequence of the `new` is also used when `init` is used as explicit method call inside a constructor.
With the new proposal, `auto` should be called instead, but because of bootstrapping issues, the existing calls to `init` are interpreted as call to `auto` instead and an advice is issued.

Manual named constructors tries to magically call a constructor if no such a call is present.
The current way to determine what and how to call it use a complex heuristic to make existing code compatible.

A in-depth review of the heuristic to rationalize the policy is required but is out of scope of the present proposal.


## User-defined `auto`

Since `auto` is compatible with standard manual named constructor, what append when the used define himself a constructor `named` auto?

TODO
