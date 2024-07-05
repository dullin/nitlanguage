You may notice that the FFI with Java is very similar to the [[FFI/FFI_with_C]]. This page highlights the difference and avoids repeating the information present in the documentation of the [[FFI/FFI_with_C]]. For this reason, it is strongly recommended that you read [[FFI/FFI_with_C]] before this page.

# Extern methods implemented in Java

The Java FFI main feature is to implement a Nit method in Java. Such a method is declared with a normal Nit signature followed by the callback declaration (more on that later), the language specified as Java, and the body. A Nit method implemented in Java will look like so:

~~~~nitish
fun foo(val: Int): Bool in "Java" `{
    // Java code
    System.out.println("Hello world from Java");

    java.util.ArrayList<Object> my_java_list;

    return var == 1234;
`}
~~~~

Let's analyse the previous segment of code.

* `fun foo(val: Int): Bool` Is the Nit signature it defines the type of the method. This dictates how it can be called from Nit but also the static types of the arguments for the implementation in Java.

  In this case, a Nit int will be a Java long and the expected return type will be a Java boolean.

* `in "Java"` indicates the implementation language, it could also be `in "C"` or `in "C++"`. See the page on the FFI with C for more information.

* The code between `` `{`` and `` `}`` is the implementation in Java. 

# Namespaces

For such a simple method as the previous `foo`, to call any services you must use the full namespace, as we did in the reference to `java.util.ArrayList`. However, to define specific importations you can add an additional Java extern blocks higher in the Nit module right after the Nit importations.

~~~~nitish
module my_nit_module

import some_nit_module
import some_other_nit_module

in "Java" `{
    // Java importations go here
    import java.util.ArrayList;
`}

# Possibly more Nit code ...

fun bar in "Java" `{
    ArrayList[Object] my_list;
`}
~~~~

# Custom Java classes

Most Java API require to implement one or many interfaces with custom Java classes. This is not the most straightforward thing to do with the FFI but it can be done in three different ways. These are normal alternatives in Java but they take a sightly different form with the FFI.

1. The easiest way, which is not always applicable, is to use anonymous classes. Creating a class from a super-class or an interface directly where it is used.

  TODO add exemples

2. You can also create a static nested class. In fact, all the Java code within a Nit module will be organized within a single Java class. You can add a static nested class to this generated class using an extern code block prefixed by `in "Java Inner"` at the module level.

3. The third alternative is to use a Java class outside of the Nit module. The annotation `extra_java_file` on the module declaration joins the class to the project. Example:

~~~~nitish
module my_nit_module is
    extra_java_file("relative_path_to_the_java_source.java")
end
~~~~

# Extern classes in Java

Nit extern classes wraps extern types to be used in Nit code. Thay allow pass data in and out of extern methods with ease. The single value of an instance of an extern class is the extern instance that it warps.

When the extern type of an extern class is in Java you get what we call a extern Java class. It is a Nit class wrapping a Java type. In Nit you can manipulate its instances pretty much like any other instances and in Java you will get a Java object.

## Declaration

An extern Java class is declared like any other extern class, however the extern type declarations must be preceded by `in "Java"`. It will thus look like this:

~~~~nitish
extern class AndroidBundle in "Java" `{ android.os.Bundle `}
	# ...
end
~~~~

To maintain the API coherence, the introduction of an extern Java class should declare two things:

* That it is subclass to JavaObject if it is not already through other super classes. 
* it should redefine the virtual type SELF with itself. This type is used by some methods of JavaObject such as...

A simple but complete extern Java class should look like this:

~~~~nitish
extern class AndroidBundle in "Java" `{ android.os.Bundle `}
	super JavaObject
	redef type SELF: AndroidBundle

	# ...
end
~~~~

## Equivalent Java type

The equivalent Java type must respect the following conditions:

* It must be described by its full namespace (ex: `java.io.File`)
* It can use the Java format or the internat format (using respectively `.` or `/` between namespaces and class names)
* Inner classes must be specified using the `$` symbol between the names of the parent class and the inner class (ex: `android.content.SharedPreferences$Editor`)

Limitations:

* Java primitive types (including the array) cannot be used as the equivalent type. You should instead use existing classes in the Java module. 
* As of now, Java generic types also cannot be used as an extern type. However, this is a planned feature.

Additional notes about the equivalent Java type:

* It can be an interface, and have methods in Nit.
* It can specialize other extern classes and Nit interfaces.

# Nit objects in Java

TODO

# Java objects in Nit

TODO

# Common use cases

## Sub-classing a Java class

Java APIs often rely on abstract classes that must be extended by the programmer. There are many ways to do so from Nit.

* Use an anonymous class.
* Create a static nested class. Example:

~~~nitish
module my_module

in "Java inner" `{
	static class MyNestedClass {
		MyNestedClass() {}

		void foo() {
			System.out.println("Hello World!");
		}
	}
`}

extern class MyNestedClass in "Java" `{ Nit_my_module$MyNestedClass `}
	fun foo in "Java" `{ return recv.foo(); `}
end
~~~

* Use `extra_java_file`

## Java threads

TODO

# jwrapper

Having read the beginning of this page you know how to wrap Java services in Nit. You may have noticed that some of the work is repetitive and that it requires a lot a boilerplate code. With this in mind, Frederic Vachon created the _jwrapper_ tool, a generator of Nit wrappers around Java classes.

## Installation and setup

1. Clone the Nit repository of Nit, install the needed dependencies and run `make` at the root.
2. Then, go to the folder at `contrib/jwrapper` and run `make`. This will create the executable, within the Nit repository, at `contrib/jwrapper/bin/jwrapper`.
3. Set the environment variable `NIT_DIR` to the path of your Nit repository. _jwrapper_ will use this variable to find existing wrappers of Java classes and automatically use them when necessary.

## Basic usage

1. Find the compile Java class to wrap, or compile it from source using `javac`. Some classes may need to be extracted from _jar_ archives. For example, classes for the Android library can be found in `platforms/android-*/android.jar` from the SDK.

2. Generate the wrapping code for the class `android.os.Bundle` with a call to:

  `jwrapper Bundle.class android_os_bundle.nit`

  This will create the file `android_os_bundle.nit` with the wrapper around the `Bundle` class. It will also define minimal wrappers around any other Java classes referenced but not already wrapped by the Nit library.

  Alternatively, you can use the `-c` option to _not_ generate those additional minimal wrappers. _jwrapper_ will instead comment the lines of code that are invalid without the referenced classes. They can be activated once the required class has also been wrapped.

3. Review the generated code and adapt it to your needs.

4. Use and import the generated module as you would with any other Nit modules. 

For more information, see _jwrapper_'s README file in your repository and [[available online|https://github.com/nitlang/nit/blob/master/contrib/jwrapper/README.md]].

# Limitations of the FFI with Java

The Java FFI has been designed with Android in mind and more specifically to be compiled against Android NDK's `jni.h`. For this reason, there is currently some limitations when targeting a desktop environment:

* The inner Java code block does not support adding inner classes.  This is a known missing feature, it can be fixed if there is a need for it. It does work when the target platform is Android. See the issue [[#769|https://github.com/nitlang/nit/issues/769]].

* There are some gcc warnings to be expected at compilation due to an incompatibility between the `jni.h` of the Android NDK and its desktop counterparts. See the issue [[#770|https://github.com/nitlang/nit/issues/770]].
