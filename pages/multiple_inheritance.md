# Multiple Inheritance

You hate copy-pasting? you like polymorphism? you like static-typing? Then multiple inheritance is for you.

In Nit, all kind of classes (including interfaces) can provide methods with bodies.
All kind of classes (including concrete classes) can specialize one or more super-class.

### But multiple inheritance is too complex for my mind, isn't it?

In Nit, inheritance (and especially multiple inheritance) is done right:

* No repeated inheritance: inheritance is only based on a principle of specialization. Therefore, a class is either inherited or is not inherited. A Cow is an Animal, there is no point to even ask "how many time a Cow is an Animal?"

* No explicit transitive specialization. If a class already specializes an other class by transitivity, explicitly declaring this class as a super-class has no effect (and a compiler might gives you a warning)

* No hidden inheritance: specialization is public, therefore if you can see the classes, you can see their specialization links.

* No static overloading: A method or an attribute introduced in a super-class is always inherited and cannot be mistaken with something different.
