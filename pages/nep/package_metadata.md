# Package Meta-data

* Status: implemented
* Discussion: <https://github.com/nitlang/nit/issues/1655>
* Implementation:
   * Association between a package and its .ini file: <https://github.com/nitlang/nit/pull/1654>
   * Filling metadata: <https://github.com/nitlang/nit/pull/1675>
   * Using metadata: <https://github.com/nitlang/nit/pull/1678>
* Preview: <http://nitlanguage.org/catalog/index.html>

In Nit, the *package* is the unit of distribution and versioning of software.
A package is made of coherent modules that are developed and should be used together.

Nit tools need information to associate a module to its packages and be able to look for dependencies inter-packages.
These information are given in a package description file, called `package.ini` located in the root directory of the package.

An example of a `package.ini` file is

~~~ini
[package]
name=foo
version=0.1
tags=algo,android
maintainer=John Doe <doe@example.com> (http://example.com/~doe)
license=GPL
[source]
include=src
[upstream]
homepage=http://example.com/foo
git=https://git.example.com/doe/foo.git
~~~

## package

The package section describe core information about the package, as listed in a catalog of packages

### package.name

The name of the package, should be a valid identifier.

By default, it is the name of the root directory.

### package.version

The version number of the package.
Free-style for the moment but will be used latter with a semantic versioning approach <http://semver.org/>

By default it is the empty string

### package.tags

Comma-separated list of tags used to categorize the package.
Tools like auto-documentation and archive networks may use them to classify the package.

By default it is the empty list

Some tags:

* ai: artificial intelligence
* algo: data structure and algorithms
* cli: command line interface
* crypto: cryptography
* database: database
* debug: debugging, profiling, testing, logging
* devel: development and software engineering
* educ: education
* embedded: embedded system
* encoding: encoding, decoding, conversion
* example: example of code, proof of concepts
* format: read, write and manipulation of file formats
* game: video games
* graphics: graphics and display
* i18n: internationalization and localization
* io: input-output
* java: Java related
* language: related to other programming language
* lib: library and frameworks
* mobile: cellphones and tablets
* network: networking
* parallelism: parallelism, threads, clusters
* platform: special platforms
* sound: sound, audio, music
* terminal: terminal, console
* text: manipulation of text
* ui: user interface
* web: web, http
* wrapper: wrapping of third-party software
* xml: manipulation of XML

### package.maintainer

List of the persons that are responsible for the package.

By default it is the empty list

Note that there is no `author` field.

### package.more_contributors

List of additionnal contributors.

The final list of contributors is computed and include the maitainers, authors collected by the vcs (`git shortlog`) and persons.
This field is expected to hold contributors thal falled trought the cracks of the vcs (e.g. non programmer contributors, bad csv history, etc.)

By default it is the empty list

### package.license

The main license of the software.
Free-style, feel free to use the identifier of license listed by SPDX <https://spdx.org/licenses>

By default it is the empty string

## source

The source section describe files in the directory of the package

### source.include

The list of directories to scan to find Nit source files.
Paths are relative to the root directory of the package.

By default it is "."

### source.exclude

The list of directories excluded from the scan of files for the package.

Unless specific conditions, there is no reason to set it.

By default, it is the empty list

### source.bin

Where executable are compiled if any.

By default, it is "`bin/`"

### source.assets

For libraries `mnit` and `app.nit`, the directory contained assets.
On some platform like `android` it is used to locate and bundle assets in the generated `.apk` files

By default, it is "`assets/`"

### sources.po

The directory containing translation files used by the module `gettext`.

By default, it is "`po/`"

## upstream

The section `upstream` groups information about accessing resources about the software and its development on the Internet

### upstream.git

An URL to the git repository of the package

By default it is the empty string

### upstream.git.branch

The git branch of the package

By default is is `master`

### upstream.git.directory

The subdirectory of the repository that is the root directory of the package.

While it is expected that each package is developed in its own repository, a package could be located is a subdirectory. 

By default is is `.`

### upstream.homepage

The URL of the public homepage of the package

By default it is the empty string

### upstream.contact

Which person, mailing list, forum, to send e-mail messages in the first place.

By default it is the email of the first author (see `package.autor`)

### upstream.issues

A URL to the list of known bugs and issues for the package.

### upstream.browse

An URL to browse the repository containing the sources.

### upstream.apk

An URL that points to a dry apk, or an install page on some store, for the main Android application offered by the package.

When set, an implicit tag `apk` is added to the list of tags.

### upstream.tryit

An URL that points to an aofficial demo version or the official live version of the main web service offered by the package.

When set, an implicit tag `tryit` is added to the list of tags.
