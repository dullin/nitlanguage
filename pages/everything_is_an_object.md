# Everything is an Object

In Nit, all data, even primitive stuff like integers and booleans, are objects.
It means :

### They are subtype of Object

~~~
fun foo(o: Object) do print "I am {o}, the {o.class_name}"
foo(4)    # -> I am 4, the Int
foo(true) # -> I am true, the Bool
~~~

### Their classes can be [[refined|refinement]]

~~~
redef class Int
    fun double: Int do return self * 2
end
print 12.double # -> 24
~~~

### And it is real

It is not just some automatic boxing (a la Java 1.5).

~~~
var a: Object = 5
var b: Object = 3 + 2
print a == b # -> true
print a.is_same_instance(b) # -> true
~~~
