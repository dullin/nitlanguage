# Conditional importation of modules

The problem, independently identified by contributors, is to allow the variation of a module according to a specific platform. Eg sound on linux vs sound on android.

* Status: Implemented
* Discussion: <https://github.com/nitlang/nit/issues/1579>
* Implementation: merged <https://github.com/nitlang/nit/pull/1580>

## Goals of the proposal

* use standard module importation and class refinement as the base mechanism
* do not require additional description of rules (nor configuration file)
* completely static and locally checkable
* not specific to platform but generalized to any module
* do not enforce a specific vendor/package responsibility but allow lib-based, platform-based and third-party based approaches (see below)
* easy to implement (in the model, in the parser, etc)

## Current Approach

Currently, the solution is to develop parallel hierarchy of modules and make that the final program imports all required combination of bottom modules (eg app.nit).
The issue with this approach is that the hierarchy of module become complex and error-prone to maintain, the second issue is that complex product line might be unnecessary non-pola to specify for a developer: "why I must import android::audio since my program already imports app::audio and is compiled with -m android?"

## Specification

* there is conditional importation rules: e.g. if all the modules m1, m2 are imported then automatically import m3
* conditional importation rules can be declared (but not applied) by any modules
* conditional importation rules are applied on clients that satisfy the conditions

The following syntax is just a syntactic base for discussion.

~~~
module foo
import bar is conditional baz
~~~

It declares a rule 'foo & baz -> bar'.
It means that in a client module of both foo and baz (direct or indirect client), the client also imports (implicitly and automatically) the module  bar.

~~~
module a_client
import foo
import baz
~~~

Conditional importations are never imported directly by the module that declares them but are deferred to the clients.
It means that a module who declares conditional importations is statically analysed (and possibly compiled) without taking in account the conditional importations nor the associated conditions.
It also means that clients are statically analyzed (and possibly compiled) after having applied all the conditional importations of its imported modules.

## Use  cases

The proposal is compatible with various use cases

### Library-side variations

A library vendor can adapt it to different platform, for instance to provide specific implementations or API extensions.

For instance, a vendor can provide library/library.nit and library/library_android.nit with the following content:

~~~
# A library with a generic API and portable implementation
module library
import library_android is conditional android
~~~

~~~
# Additional services and specific implementation of library on android
module library_android
import library # because we refine it
import android # because we rely on Android service and api to provide the additional things
~~~

### Platform-side variations

In a symmetrical way, a platform vendor can choose to provide extensions and specific implementation to common libraries that will be automatically used on the platform.
It could be used to redefine method with a functional (or a more efficient) implementation.

~~~
# general platform module to compile for android
module android is platform
import android_jvm is conditional jvm
~~~

~~~
# special implementation to access the jvm on Android
module android_jvm
import android_base
import jvm
~~~

### Third-party-side variations

A last use case is when two libraries could be extended if used together but none of these provide a conditional importation of the other. In these case, a third-party vendor can provide a auto-glue library made of conditional importations.

~~~
# glue to partially implements libraries of app.nit for the emscripten platform
module appjs
import appjs_sound is conditional emscripten, app::sound
import appjs_storage is conditional emscripten, app::storage
~~~

## Issues

* What syntax
* What about the visibility of the automatic imported modules
* What about linearization rules of the automatic imported modules
