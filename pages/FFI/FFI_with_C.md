# FFI with C

The Nit Foreign Function Interface (FFI) allows to nest C code within a Nit file. Doing so, you can implement Nit methods in C, declare the C type equivalent to a Nit class, and write supporting C code (such as imports). This page is an introduction to different aspects of the Nit FFI.

Common use cases of the FFI is to optimize a method or wrap existing C libraries. In case of a wrapper, the C code manages calls to the C library functions and callbacks to Nit. To know more about wrappers using the FFI, see the page [[FFI/Wrapping_C_libraries]].

# Extern methods

An extern method is a Nit method implemented with C code. Except for its implementation, it behaves like any other Nit method. It has a receiver, can refine or specialize an existing property, be refined itself and even have virtual types in its signature.

Extern methods are declared and implemented like so in the Nit code:

    class A
        fun foo `{
            printf( "in method A::foo, from C\n" );
        `}
        fun bar( n : Char ) : Bool `{
            printf( "in method A::bar( %d ), from C\n", n );
            return 1;
        `}
    end

Notice that the body of the previous methods are implemented in C. This code behaves as a C function with a hidden signature. For the previous methods, the hidden signatures are:

~~~c
void A_foo___impl( A recv )
int A_bar___impl( A recv, char n )
~~~

The Nit types considered as primitive to C are converted in the C signature (as for Char and Bool) whereas Nit types unknown to C are kept opaque (as for the receiver of type A).

# Primitive Nit types in C

For easier passage between Nit and C, Nit types that are primitive to C are converted automatically. This conversion works both ways.

## Primitive types between Nit and C

~~~pandoc
| Nit type    | C type | Notes |
|:------------|:-------|:------------|
| `Int`       | `long`   | The normal primitive type in C |
| `Bool`      | `int`    | Where 0 is false and everything else true |
| `Float`     | `double` |  |
| `Char`      | `uint32_t` |  |
| `Byte`      | `unsigned char` |  |
| `NativeString` | `char *` | The standard C string |
| `Pointer`   | `void *`    | A general pointer |
~~~

## Note on NativeString

`NativeString` is a pointer, it may point to memory managed by the garbage collector.
However, apply special care when dealing with `NativeString` instances from C,
especially to avoid memory leaks or painful-to-debug segmentation faults.
To avoid any problems when dealing with `NativeString`, follow these basic rules:

* If you don't know the origin of a `NativeString`, it may have been allocated by the GC.
* Be particularly careful when modifying a `NativeString` as they might be shared by several instances,
  a general rule would be not to modify only copies of the received `char*`.
* When returning an extern resource, allocate a `NativeString` managed by the GC through `new_NativeString` in C or use `NativeString::to_s_with_copy`.

# Nit types in C

All Nit types that are not primitive to C are represented by an opaque C type. _The opaque C type is generated when needed with a name corresponding to the Nit type_. Usually the C type will have the same name as its Nit equivalent, with the exception of nullable types (in which the type name is prefixed with _nullable\__) and generic types (see examples below).


## Some equivalent types between Nit and C

~~~pandoc
| Nit type    | C type | Notes |
|:------------|:-------|:------------|
| `Object`    | `Object` | A generated opaque type |
| `String`    | `String` | A generated opaque type (`NativeString` is more practical in C) |
| `nullable Object` | `nullable_Object` | Using casts (discussed later on this page) it can be cast to `Object` |
| `Array[String]` | `Array_of_String` | There is a `_of_` between the class name and it's parameter. |
| `Map[Int, String]` | `Map_of_Int_String` | The parameters are separated only with a `_` |
| `nullable Array[String]` | `nullable_Array_of_String` | These can get quite long, but they are precise! |
~~~

# Callbacks to Nit from C

As seen previously, to invoke C code from Nit code, you must use extern methods. But when calling Nit code from C, you must use the callback system of the Nit FFI.

Any callbacks from C to Nit must be declared after the signature of the extern method. In fact, they must be declared on the last Nit extern method on the stack. With this information, the compiler will generate a custom C function implementing the callback and will make it available to the C code implementing the last Nit extern method.

The generated functions are named using the receiver type and the method name.

    class A
        var my_attr = 1234

        fun baz( msg : String ) import String.length, String.to_cstring, my_attr, my_attr= `{
            char *c_msg;
            int msg_len;

            /* String_to_cstring is a callback to msg.to_cstring */
            c_msg = String_to_cstring( msg );

            /* String_length is a callback to msg.length */
            msg_len = String_length( msg );

            printf( "received msg: %s, of length = %d\n", c_msg, msg_len );

            /* A_my_attr is a callback to the getter of self.my_attr */
            printf( "old attr %d\n", A_my_attr(recv) );

            /* A_my_attr is a callback to the setter of self.my_attr= */
            A_my_attr__assign( recv, msg_len );
        `}
    end

As you may have noticed, the callback functions in C use a similar name to the equivalent Nit method and the implementation functions. This pattern can be used to predict the name of the C function to called a given Nit method.

It is also possible to callback the Nit language for more than methods:

* Declaring an import of `super` in Nit will allow to call super from C using a function similar to `A_baz___super( recv )`.
* Declaring an import a constructor will allow it to be called from C as `A new_A()` for the anonymous constructor of the class A or `A new_A_name()` for a named constructor.
* Casts can de declared as an import in Nit with three forms; `A as(B)`, `String as nullable`, `String as not nullable`. The first form is used to cast from any to any type. The second and third forms are used to cast between nullables and non-nullables. In the C code, those imports allows to use two set of funtions, one for the type check and one for the cast itself. With these examples we would get `int A_is_a_B( A )` and `B A_as_B( A )` for the first form; `nullable_String String_as_nullable( String )` for the second form (no check is necessary) as well as `int String_is_null( nullable_String )` and `String String_as_not_null` for the third form.

## Callback Formats by Examples

We will use examples in order to familiarize you to the structure generated callback functions. But first, we setup the context in the next code segment. Then, we explain the callbacks from `my_c_meth`.

### Example Context

    class MyClass
        var attr: Object
        var attr_int: Int

        init(obj: Object, val: Int)
        do
            self.attr = obj
            self.attr_int = val
        end

        init default
        do
            self.attr = "Hello"
            self.attr_int = 8
        end

        fun foo do print "foo"
        fun bar(text: String, val: Int): Bool do return text.is_numeric and text.to_i == val

        fun my_c_meth import foo, bar, attr, attr=, attr_int,
                             attr_int=, MyClass, MyClass.default,
                             NativeString.to_s `{
            // C implementation code
        `}
    end

### Explanation of each Callbacks

~~~pandoc
| Importation in Nit | C Function Signature | Note |
|:-------------------|:---------------------|:-----|
| `foo` | `void MyClass_foo(MyClass)` | The single argument is the receiver |
| `bar` | `int MyClass_bar(MyClass, String, long)` | One receiver, two parameters and a return. |
| `attr` | `Object MyClass_attr(MyClass)` | Getter of attribute `attr` |
| `attr=` | `void MyClass_attr__assign(MyClass, Object)` | Setter of attribute `attr` |
| `attr_int` | `long MyClass_attr_int(MyClass)` | Getter of attribute `attr` |
| `attr_int=` | `void MyClass_attr_int__assign(MyClass, long)` | Setter of attribute `attr_int` |
| `MyClass` | `MyClass new_MyClass()` | A call to the anonymous constructor |
| `MyClass.default` | `MyClass new_MyClass_default()` | A call to the constructor `default` |
| `NativeString.to_s` | `String NativeString_to_s(char*)` | Converts the primitive type `char*` to the Nit class `String` |
| `String.to_cstring` | `char* String_to_cstring(String)` | Converts the primitive type `char*` to the Nit class `String` |
| `Int.+` | `long Int__plus(long, long)` | Operators such as `+` use a literal name prefixed by `_` |
| `Container[Int].item=` | `void Container_of_Int_item__assign(Container_of_Int, long)` | Add to a generic array bounded by String |
| `Array[String].add` | `void Array_of_String_add(Array_of_String, String)` | Add to a generic array bounded by String |
| `Array[String].[]` | `String Array_of_String__index(Array_of_String, long)` | Get from an array. |
| `Array[String].[]=` | `void Array_of_String__index_assign(Array_of_String, long, String)` | Insert into an array. |
| `HashMap[String, Array[String]].[]` | `Array_of_String HashMap_of_String_Array_of_String__index (HashMap_of_String_Array_of_String, String)` | When the types are too complex like this one, you are better of to write the complex code in Nit. |
~~~

# Extern classes

A Nit extern class is a special kind of class that are associated to a custom C type. An instance of this class can then be used like other primitives when passed to C where its equivalent type will be used.

An extern class is declared in Nit using the `extern` keyword before `class`. The C type equivalent is defined in C right after the class name.

    # A Nit extern class associated to the C type `my_struct *`
    extern class MyStruct `{ my_struct* `}
        # ...
    end

The single value of an instance of an extern class is its value in C, which means that it has no attributes. An extern Nit classes still can have methods (normal and extern) and constructors (but only extern constructors, presented shortly).

In total, extern classes impose 3 restrictions:

* The extern class cannot define attributes.
* The equivalent C type _must_ be a pointer (it is actually a subclass of `Pointer` in Nit)
* The extern class can only subclass other extern classes and interfaces.

Instances of extern classes can originate from the C code, or from an extern constructors. Extern constructors are declared using the _new_ keyword and are implemented similarly to extern methods. They must return a value in the equivalent C type.

    extern class MyStruct `{ my_struct* `}

        # Extern anonymous constructor doing a simple malloc
        new `{
            return malloc( sizeof( my_struct) );
        `}

        # Named extern constructor a bit more complexe
        new named `{
            A a = malloc( sizeof( my_struct) );
            (*a).age = 21; // this wiki do not support the standard arrow
            return a;
        `}
    end

You _must_ free any resources you allocate in C code, the Nit system do not manage these.

## Examples of extern classes types in Nit and C

~~~pandoc
| Nit type     | C type        | Notes |
|:-------------|:--------------|:------------|
| `MyStruct`   | `my_struct *` | As declared by the user (in the example above) |
| `NativeFile` | `FILE *` | A handle to a file, from the standard library |
| `FileStat`   | `struct stat *` | Informations on a file, from the  standard library |
~~~

# Global C references

Nit objects passed as argument to a C implementation function are guaranteed to be valid until the function return. So it is for Nit objects returned from callbacks to Nit code. However, if you need to keep a Nit object between functions, in a global C variable, you have to notify the native interface for it to sta valid.

For example, before saving an instance of the class A in a global variable, you must call `A_incr_ref( A )` on the instance. From then on, the Nit system will update this reference at each garbage collection so it stays valid, even after the current extern method returns. To free such a reference, call `A_derc_ref( A )`.

# Compiling and linking

Compiling a Nit program using the basic FFI is straight forward:

~~~sh
nitc my_module.nit
~~~

When using a shared C library you will have to specify more options for the compiler.

## Passing options to the C compiler

For advanced use of the FFI, you'll often need to pass custom options to the C compiler. The most common are probably: `-l` to link with a shared library and `-I` to specify where to find C header files.

The Nit FFI offers 3 annotations to use on the modules nesting C code: `c_compiler_options`, `c_linker_options` and `pkgconfig`.

### pkgconfig

The program `pkg-config` knows what options are needed to compile a specific with popular C libraries. The annotation `pkgconfig` call the program in the background to get the options.

Usage examples:

    module glesv2 is pkgconfig # Will use the options for the name module `pkg-config glesv2`

or

    module gtk3_4 is pkgconfig("gtk+-3.0")

### cflags and ldflags

When you need specialized options or a library unsupported by pkg-config, you can use the annotations `cflags` and `ldflags`. They pass options directly the compiler and linker.

They can be used with a special function call to `exec` which will use the result of a local program instead of a string.

Basic usage example:

    module sdl is
        cflags("-I", "/usr/include/SDL")
        ldflags("-l", "SDL")
    end

Same example with calls to `exec`, where we use the external program `sdl-config`:

    module sdl is
        cflags(exec("sdl-config", "--cflags"))
        ldflags(exec("sdl-config", "--libs"), "-lSDL_image -lSDL_ttf")
    end

<!---
# Example using the FFI

The example extern_methods.nit illustrates a normal use of the Nit FFI.

* [[examples/extern_methods.nit|http://nitlanguage.org/nit.git/blob/HEAD:/examples/extern_methods.nit]] (not yet updated)
--->

# References
* [[L'interface native de Nit, un langage de programmation à objets; Alexis Laferrière; Mémoire; UQAM |http://xymus.net/alaferriere-memoire.pdf]]
