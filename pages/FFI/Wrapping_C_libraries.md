TODO common use cases

# FFI

Wrapping a C library relies mostly on the Nit FFI and for this reason understand this page requires a basic knowledge of the FFI. You can refer to [[the manual of the FFI|FFI/FFI_with_C]] to learn the features and terminology of the Nit FFI.

# Wrapping C Functions

In order to provide access to a C function in a Nit API or module, the first step is to wrap the function behind a Nit method. The recommended approach is to use a one to one mapping when possible.

For example, to wrap the C function `sleep` we create an extern Nit method with a similar name. It is recommended to use a name that can be recognized by a user with previous knowledge of the C library. Our extern method relies on the automated conversion of Nit primitives for the FFI.

    fun sleep( t : Int ) `{ sleep( t ); `}

Since every Nit method has a receiver, it is recommended to use it and attach the extern method to a relevant class. In this case, we attach our method to the Int class by refining it.

    redef class Int
        fun sleep `{ sleep( recv ); `}
    end

Here we use the Int class, as a FFI primitive, its instances are automatically converted to `int` in the C code. In the next section, we will see how to define our own FFI primitive classes.

Besides the primitive types of the FFI, we often need to use the `String` type. Only the `NativeString` type is converted to `char*`. When writing the low-level layer of the interface wrapper, you can use the `NativeString` to simplify the code. Higher-level layers can often convert from `String` to `NativeString`. Otherwise, it is also common to declare the callback to `String::to_cstring` and `String::from_cstring` to manage the conversion from the C code.

## Nity and Native Methods Pattern

When the C method expects many arguments, lets say many `NativeString`, you can use an extra indirection method to convert `String` into `NativeString`. We call this indirection method _Nity_, in comparaison to the _native_ method in C.

~~~
# The Nity indirection method
fun sdl_warning_box(title, content: String)
do
    native_sdl_warning_box(title.to_cstring, content.to_cstring)
end

# The native C method
private fun native_sdl_warning_box(title, content: NativeString) `{
    SDL_ShowSimpleMessageBox(SDL_MESSAGEBOX_WARNING, title, content, NULL);
`}
~~~

The native method method can be marked as private since the API should expose only the Nity method. Of course, the returned value can also be converted to Nit in the Nity method.

This pattern also applies when calling a Nit method within a normal class and which access instance attributes. An indirection method can gather these attributes and pass them to the native method.

# Wrapping C Structures

Extern classes define a Nit class which only value is available in C. These can be used to wrap a C structure in a C class.

We recommend to use an extern class to wrap all interesting C structures, even the small ones. It allows to correctly classify the wrapped functions and manipulate them from Nit.

The main C structures that are the most important for the library are often strongly linked to some functions. In this case, we define the extern class to represent the structure in Nit, and use it to encapsulate all the strongly linked wrapped functions. You can often identify such cases in C when a set a functions use the same data structure as the first parameter (or sometimes last parameter). These funtions are also often prefixed with the name of the structure or a common namespace.

When such a structure and function set is identified, we build a an extern class associated to the C structure in multiple steps:

1 Name the extern class in a way that it can be recognized. It is often possible to reuse the name of the structure for the class. You may want to remove the namespace-like prefix from the structure name as those are managed by Nit.

2 Specify the C type equivalent to the extern class. In this case, the C type equivalent would be a pointer to our structure (the equivalent type must be a pointer or compatible with a pointer).

3 For every C function creating this structure, add an extern constructor to the extern class. Extern constructors is the only point where it is possible to et the value of the instances of the class from within the class. Note that, it is "normal" for an extern constructor to fail, in such cases we use an additional method to verify if the instances are valid.

4 Create an extern method for each other C function strongly linked to the structure. As a rule of thumb, these functions can often be identified by their use of a common prefix or the use of the structure as the first parameter.

The result of these step fon an extern class might look like so:

    extern class Curl `{ CURL * `}
        new `{ return curl_easy_init(); `}
        fun is_ok `{ return recv != NULL; `}
        fun cleanup `{ curl_easy_cleanup( recv ); `}
    end

# Dealing with C enums


Though not ideal, as of now it is possible to represent a C enumeration in Nit by using an extern class. The extern class is associated to the type, as usual and we can use constructors and extern methods to manipulate the fields of the enumeration.

We will use the libCURL enumeration `CURLoption` as example. In this case, we would declare the header of the extern class as so:

    extern class CURLoption `{ CURLoption `}
        # ...
    end

You may notice in the previous example that the extern type is not a pointer type. In this case, the enumeration type is still compatible so we can use this approach.

To obtain a value of the enumeration from Nit, we define extern constructors. We usually need one constructor per value, all it does is return the C value. In our example, we define the constructor `verbose` to return the value `CURLOPT_VERBOSE`, it can then be obtained using `var x = new CURLoption.verbose`.

    extern class CURLoption `{ CURLoption `}
        new verbose `{ return CURLOPT_VERBOSE; `}
    end

In order to verify the value of an instance of our extern class, we define additional methods. In our example, to check whether an instance of `CURLoption` is a `CURLOPT_VERBOSE` we add a method to our class:

    extern class CURLoption `{ CURLoption `}
        new verbose `{ return CURLOPT_VERBOSE; `}
        fun is_verbose `{ return recv == CURLOPT_VERBOSE; `}
    end

# Dealing with callbacks

C libraries commonly use function pointers to callback the user code. Some libraries use this to notify the program of available data, execute tasks in the background or react to events. They will usually use function pointers with some sort of user data.

There is no direct equivalent to function pointers in Nit. However there is an object oriented way to achieve the same goals.

Let's reuse the libCURL example, it uses function pointers to notify the user code of data availability. The user code then manages the data, write it to disk or keep it in memory.

In this case, we create a class with a single abstract method. We will name the class `CurlCallable` and the method `curl_callback`. We will also create a C function called `my_module_curl_callback` and use it as target for all function pointers. This function retrieves the real targeted instance of `CurlCallable` stored in the user data and calls its method `curl_callback`.

The user code creates a subclass of `CurlCallable` to receive the calls. It is then passed as the user data when setting the callback. The reference count of the instance must also be incremented to stay valid until the actual callback, with `CurlCallable_incr_ref`.

TODO Review this section when adding an example program using this technique in the example folder.

# Handling errors

Each C library has a way to reports errors, but there is no standard. It is common to use a global variable to store error code (ex: _errno_) or to directly return a code from each functions.

There is no standard way to manage errors in Nit either. For this reason, it is recommended to reuse the system from the C library in the Nit module. This creates a wrapper without surprise for users familiar with the C library. It is also usually much easier and cleaner to implement!

# Dealing with pseudo inheritance of C structures

TODO

# Import C header files

When wrapping a C library, you usually need to import a C header to access its signatures, types and macros. This importation is done in an extern block after the module declaration like so:

    module my_module

    `{
        #include <curl.h>
    `}
     # ...

By default, this block is intended for the header of the C body, which covers all C implementation code. However, if you have public extern class in your module, the importation must be declared in the  C header code. For wrappers it is almost always the case.  To do so, the previous declaration would look like so:

    module my_module

    in "C Header" `{
        #include <curl.h>
    `}
     # ...

# C functions using parameters to return values

TODO

Basically, use the Container class or create your own.

# General architecture

As general architecture of a wrapper we recommend to use two layers. The first layer, we will refer to it as the native layer, uses the FFI to connect Nit to C. The second layer, the abstraction layer, provides an object oriented API that hides partially the interface layer from the user.

## The native layer

The native layer uses extern methods, extern classes and all other features of the FFI to bridge Nit and C. We won't discuss much more of the native layer in this section as the rest of this manual describes every aspects and tricks of this layer.

##  The abstraction layer

The abstraction layer aims to offer a Nity API to access the feature of the wrapped library. The idea of abstraction is applicable to any libraries, not only wrapped C libraries, so in this section we discuss only of the aspects relevant to a wrapped C library.

The most important aspect of the abstraction layer when wrapping a C library is to hide unusual Nit classes and types from the user. For instance, a normal Nit user unfamilliar with the FFI might attempt to specialize an extern class to extend it with a new attribute, which is not possible. This user will b surprised to manipulate a class from The API without being able to adapt it to its needs. For this reason, the abstraction layer often provides normal Nit classes acting as a thin wrapper around an extern class. This allows the user to manipulate classes from the API as any other classes and, as a bonus, often helps with complex uses of the FFI.

The abstraction layer can be more or less abstract and even offer different levels of abstraction. It is common to have a total of three layers: the interface layer, the specific Nity API and the high-level abstraction layer. Each of these can be made accessible to the user, there is no need for only one to be public, then the user might choose the most appropriate layer/API according to her needs.

## A practical example

Let's take as example a wrapper for the cURL library (libcURL). Note that it may not represent that actual structure of the Nit _curl_ module. It could be using the three layers:

* The native layer wraps every libcURL data structure in an extern class and creates a one-to-one mapping of the C function as Nit methods. It is not Nity but can still be used by users familiar with the FFI and the wrapped libcURL.

* The specific Nity API encapsulates properties of the interface layer if logicial Nit classes. It offers the HTTPRequest class which manages most of the basic request building and requires the user parameters as argument to its constructor. The HTTPRequest provides services to execute the request, store the results and identify possible errors. Using the three layer idea, this class would remain very linked to the underlying interface layer and libcURL. For instance, its error codes are those of libcURL and the format of parameters  is the same as libcURL.

* The high-level abstraction layer is to be used in a script or a prototype program. The user does not care about the underlying implementation and want only minimal error management. This layer will mostly define functions in existing classes to integrate seamlessly in the existing library. For example, the _curl_ module might add `String::download_to(path : String) : nullable String` which downloads the file identifier by the URI of the receiver String and returns the error message in case of error.

There is an additionnal layer, or an alternative to the high-level abstraction, possible using the same example of libcURL.

* The pure abstraction layer would offer an implementation agnostic API, or reuse an existing one, and implement it with the specific Nity API. The advantages of such an interface is that the implementation may change (due to a difference in architecture and the evolution of the underlying layers and library) without modifying the user code. A good example of such a layer is the C/Java library ODBC.
