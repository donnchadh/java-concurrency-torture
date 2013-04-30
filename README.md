# JAVA CONCURRENCY TORTURE TESTS

This project is the attempt to test/break Java implementations 
on concurrency front. This harness can be used to conduct the fine
experiments which require lots of art to expose otherwise.

## Quick-start:

```
 $ mvn clean install
 $ java -jar application/target/concurrency-torture.jar -h

```

## Caveats:

* Most of the tests are probabilistic, and require substantial time
  to catch all the cases. It is highly recommended to run tests longer
  to get reliable results.

* Most of the tests require at least 3-4 online CPUs. Low CPU count
  machines could also use these tests, but always enable yielding
  there. Consult the command help to figure out the option for it.


## Understanding tests and Interpreting results:

The tests so far are folded in Litmus-like tests, where few threads
are  executing the test concurrently, sometimes exhibiting races.
There  are multiple state objects generated per each run. Threads then
either mutate or observe that state object.

Test harness is collecting statistics on the observed states. In many
cases this is enough to catch the reorderings or contract violations
for concurrent code.

The console output can be used to track progress and debugging.
Ordinary users should use results/index.html, which has the full
interpretation of the results.

## Developing tests:

If you want to develop a test, you are encouraged to get familiar with
existing set of tests first. You will have to implement one of the
test interfaces:
- OneActorOneObserverTest
- TwoActorsOneArbiterTest


Read up their Javadocs to understand the conditions that are guaranteed
for those tests. If you need some other test interface/harness support,
please don't hesitate to raise the issue and describe the scenario you
want to test.

Test interpretation is handled elsewhere, you can see at 
`application/src/main/resources/net/shipilev/concurrent/torture/desc/`

Each XML file there describes the test suite, which is the collection of
tests. It can match the output for each test and mark if that state is:
- *REQUIRED*:         should always be present
- *ACCEPTABLE*:       may be present, may be absent
- *FORBIDDEN*:        should always be absent
- *KNOWN_ACCEPTABLE*: may be present, special known case
- *KNOWN_FORBIDDEN*:  should always be absent, special known case


KNOWN_* are useful to document known corner cases and/or building the
negative tests, they are treated as their counterparts, but routinely
highlighted in the reports.

You are encouraged to provide the thorough explanation why particular
state is required/acceptable/absent/special. Even though harness will
print the debug output into the console if no description is given.

Tests may be created in a separate project. To accomplish this, include
the following dependency in your project's pom.xml:
```
	<dependency>
		<groupId>net.shipilev</groupId>
		<artifactId>concurrency-torture-api</artifactId>
		<version>1.0-SNAPSHOT</version>
	</dependency>
```
From there, the `net.shipilev.concurrent.torture.api` interfaces may be
imported and implemented. Along with each test, an XML description file
should be included (matching the format described above). To run tests 
written this way, rename to your jar file to `tests.jar` and place it 
alongside the concurrency-torture jar file. This will add the jar to the 
classpath so it can be scanned for tests. The next step is to specify 
`-package` and `-desc` parameters when running the jar:
```
$ java -jar concurrency-torture.jar -package 'your.test.package.*' -desc 'your.description.package'
```
