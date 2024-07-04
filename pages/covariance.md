# Covariant Typing Policy

Covariance in Nit has two causes:

  * generic types are covariant on the base type: ie `Array[Cat]` is a subtype of `Array[Object]` because `Cat` is a subtype of `Object`. This is not true in most of object-oriented languages.
  * [[virtual types]] can be redefined

## Advantages and drawbacks of the covariance

The main advantage of the covariance is the improved expressiveness to the Nit models.

~~~
class Employee
    type OFFICE: Office
    var office: OFFICE
end

class Boss
    super Employee
    redef type OFFICE: BossOffice
end

var b: Boss = ...
var o: Office = ...
b.office = o # Compilation Error! Expected a BossOffice
~~~

Invariant languages (eg. Java, C++) have to use unsafe downcasts to achieve the same effects.
