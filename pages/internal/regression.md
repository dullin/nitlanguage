# Regression Tests

The directory `tests` contains various small Nit programs used to check that the compiler is not broken.

* [[tests/README|http://nitlanguage.org/nit.git/blob/master:/tests/README.md]]

To run the full regression test-suite, run:


~~~sh
cd tests
make
~~~

Individual tests can also be run independently:

~~~sh
./tests.sh some_nit_programs.nit...
~~~

Each time some feature is added by a developer, he should:

* bootstrap the compiler in src/ : `cd src/ ; ./ncall.sh`
* run the full regression test-suite : `cd tests/ ; make`
* add new tests to prevent future breakage of the new shiny feature

