# Testing


Developers are responsible for unit testing.  Generally by 'unit', I mean any
given subset of the application under test, as small as practically possible.
This is often a single class, but not necessarily.  For example, a Controller
may be unit tested, even though the 'unit' being tested is actually a graph of
objects.  Keep the graph as small as practically possible, but large enough to
exercise a meaningful API boundary.

## Types of Testing

* unit ( no dependencies at all, only in-memory object graph allowed.  No
  network, or filesystem access)

* quick (local services only, mysql, mongodb, etc.  Mocked dependencies also 
  allowed).  No remote service calls allowed.

* tests that may be run either in "quick"  OR "full" integration.  

* true integration -- hitting "non-local" endpoints ONLY.  NO mocking / local
  services allowed, unless in very specific and presumably rare circumstances.


## Directory Structure

test/unit -- unit tests without dependencies

test/integration -- quick and full integration tests both.  

test/web-api -- same as test/integration but focused entirely on the REST api
rather than lower in the stack.  

Tags (via Annotations) will specify in the test what mode it can run in (what dependencies it requires and what dependencies it cannot use.)


## Tags and Modes

A tag (expressed via an Annotation) indicates what environment an integration
test may run in.  Tags determine dependent services that are injected into the
envirionment.  Legal values for the tag annotation are:
'MockedIntegration', 'MockedOnlyIntegration', or 'Integration'



A mode determines what the goal of integration testing is:

* 'quick' for all tests that can execute with mocked or local dependencies

* 'full' for all tests that can execute with real remote services.

## Tagging

Specs should be tagged with annotations to designate their intended use.  

This annotation is on the Class and would look like:

```scala
@MockedIntegration
class MyExampleSpec extends SungevitySpec {
...
}
```

The following annotations should be used:

### MockedIntegration Annotation

This spec may run both with mocked dependencies OR with real, external,
"production" dependencies.  

Fixture data should comply with the rules outlined below, in section Fixture Creation Rules.

### MockedOnlyIntegration Annotation

This spec indicates that it is only inteded to run against local services.  Good
candidates for this kind of annotated spec are those that:

* would be extremely slow in real life

* would have undesirable side-effects, such as deletion of data, email sending,
  etc.

However, since it is not actually able to run against "real" environments, it
should be a type used sparingly.  It can only prove integration against a local
environment where differences in data may be relevent.

### Integration Annotation

This spec indicates that it is ONLY appropriate to run on a "real" envirornment.
Specs that may have this use-case would include:

* dependencies on salesforce or another non-local service where Betamax cannot be used to simulate it.

## Fixture Creation Rules

If a spec is tagged as MockedIntegration, it implies that it is somewhat data
agnostic between local services and "real" (Dev/QAE/UAT/Prod) services.
Therefore, fixtures should return data derived from production data when that
data is used to assert against.  This enables the spec to run against both local
services and remote services without modification.

The usefulness of this is the following:
* Quick mode proves that integration works given a particular contract.  If
  service <X> returns data <Y> in a certain format, we prove that we can handle
  it correctly.  It is also fast.  It can not, however, prove that service <X>
  *actually* behaves that way.

* Integration mode proves that the service *actually* behaves in the way you
  expect.  It is, however, slow.

## Spec Inclusion Rules

Since we support both ScalaTest and specs2 specs, a framework-agnostic means to
filter in desired classees would be preferable than implementing such a filter
for both frameworks.  

Specs inclusion will be done within sbt, via something like the following:

```scala
testOptions in IntegrationTest := Seq(Tests.Filter(s => SungevitySpecFilter.shouldInclude(s)))
```

The SungevitySpecFilter will depend on an integration test 'mode', which
determines what type of integration testing we are doing.

Each "mode" will be enabled by passing in an system property via -D flag, such
as '-Dsungevity.integration.mode=full'

Legal values for mode will be 'full' or 'quick'.  The default is 'quick'

Once the mode exists, a SungevitySpecFilter class will filter classes according
to the following:


Where SungevitySpecFilter does the following:

* Check configuration mode -- one of 'quick' or 'full'

* If 'quick', select any specs with Annotations of MockedIntegration or 
MockedOnlyIntegration

* if 'full' select any specs with Annotations of MockedIntegration or
  Integration

[ClassUtil](http://software.clapper.org/classutil) may be useful here, however out of the
box, it is compiled against against a different version of scala, which causes 
class incompatability issues with our codebase.  It can, however, be compiled from source.


## Dependencies

* mongodb -- local with fixtures or remote

* mysql -- local with fixtures or remote

* salesforce -- mockable via Betamax 

* pricing engine -- local with fixtures or remote mysql access

* document mgr -- unknown 

## Tools

* [Betamax](http://freeside.co/betamax)

* [Classutil](http://software.clapper.org/classutil)


# Mocks vs. Fixtures

Maintaining mocks can become unweildy.  Generally, it's preferable and stronly
advised to favor fixtures whenever possible.  Cases in which mocking is required
are, for example, salesforce.  Salesforce should be mocked using Betamax (http://freeside.co/betamax)
Other dependencies should be loaded via fixtures and run either in-memory or as
a stand-alone local service.

## Mocking

Mocking will be required for salesforce may be required for the document manager
Betamax allows capture / playback of HTTP traffic transparently.  

## Betamax 

The below example may change.  The betamax tape scope may be at the class level and be mixed
in automatically per class.  ie: any requests for the test class will be proxied
through betamax.  The down-side to this approach is that there is only a single
betamax "tape" (data file), rather than potentially one per example.

A betamax-enabled spec looks like the following:
```scala
    "return installation data if user is authenticated" in Betamax("install data") {

      val installationData = controller.getUserWithReferralCodes(userid)(FakeRequest(GET, s"/v1/user/$userid")
        .withHeaders("Authorization"->s"Bearer $nonAdminTknStr")
      )

      status(installationData) must equalTo(200)
    }
```

Any requests that Play issues are routed through a proxy.  That proxy looks for
pre-recorded requests that match the URL being hit.  If such a request has been
recorded, a canned response is loaded from a YAML file and passed back to the
caller.

This YAML file would be committed as fixture data along with new tests that
require it.  

The following code can be used to enable betamax with both specs2 and ScalaTest:

### specs2

```scala
  class Betamax(tape: String, mode: Option[TapeMode] = None) extends Around {
    def around[T: AsResult](t: => T) = Betamax.around(t, tape, mode)
  }

  object Betamax {
    // syntactic sugar does away with 'new' in tests
    def apply(tape: String, mode: Option[TapeMode] = None) = new Betamax(tape, mode)
  
    def around[T: AsResult](t: => T, tape: String, mode: Option[TapeMode]) = {
      synchronized {
        val recorder = new Recorder
        val proxyServer = new ProxyServer(recorder)
        recorder.insertTape(tape)
        println("Tape inserted...")
        recorder.getTape.setMode(mode.getOrElse(recorder.getDefaultMode()))
        println("tape is writable?" + recorder.getTape.isWritable)
        println("tape is readable?" + recorder.getTape.isReadable)
        proxyServer.start()
        println("proxy server started.")
        try {
          AsResult(t)
        } finally {
          recorder.ejectTape()
          proxyServer.stop()
        }
      }
    }
  }
```

### ScalaTest

```scala
  // Supports the notation test("foo") _ using betamax("tape name") { ...test body... } for ScalaTest specs
  trait Wrapped {
    implicit def wrapPartialFunction(f: (=> Unit) => Unit) = new wrapped(f)
  
    class wrapped(f: (=> Unit) => Unit) {
      def using(f1: => Unit) = f {
        f1
      }
    }
  }

  trait Betamax extends Wrapped{

    def betamax(tape: String, mode: Option[TapeMode] = None)(testFun: => Unit) = {
      println("Starting Betamax")
      val recorder = new Recorder
      val proxyServer = new ProxyServer(recorder)
      recorder.insertTape(tape)
      recorder.getTape.setMode(mode.getOrElse(recorder.getDefaultMode()))
      proxyServer.start()
      try {
        testFun
      } finally {
        recorder.ejectTape()
        proxyServer.stop()
      }
    }

```
## Fixtures

Details here are still being worked out in terms of loading fixtures to mysql
and mongodb.  Some baseline(s) will be available and can be loaded by a sbt task
for the following services:

* mysql via mysqldump

* mongodb 

* pricing engine 


## Chain of Events in Constructing Specs

1. sbt -Dsungevity.integration.mode=full it:test

2. sbt specifies the play application configuration based on the integration mode.  This determines the endpoint URLS, etc.

3. The base SungevitySpec will extend BeforeAndAfterAll and do any general setup.  Reserved for future use.

4. Any fixture traits mixed into the test are loaded upon beforeAll()

5. The tests run.

6.  Fixture are torn down in afterAll()


## Environmental Goals

Speed is a key motivation to all this.  Staying within the sbt console allows
incremental compilation and setup and should always be possible *if* possible.

## Running Tests

Tests are differentiated by configuration params passed to the runner.  

A configuration describes whether the runner(s) should select:

* only quick tests

* only "full integration" tests 

* integration tests AND tests that may run in either


## SBT Tasks

### sbt test

Test will run unit tests only.  Unit tests should have NO dependencies at all,
as opposed to integration tests.  If you need anything other than an in-memory
object graph, it is NOT a unit test.

### sbt it:test

Will run integration tests from the IntegrationTest sbt configuration scope.  

## Configuration

Test play configuration will extend to include:

* conf/application_test_itquick.conf

* conf/application_test_itfull.conf

Each of these will be selected by sbt based on the *mode* of testing and switch
the endpoints from local to external services.

## Database Fixtures

These are tests that run only against local and/or mocked services, and generally dependent on some kind of stored data and/or event playback.

Inititial db state can be set up through the use of db test fixtures, which will load the local database to a known baseline, or through use of an in-memory database.  If complex queries (slick or SQL) are involved, it is recommended that the same database driver should be used as will be used when deployed. (i.e. local mysql is preferrable to h2 running in mysql mode)

If an existing test fixture requires extensive modification for the initial state, then consider creating a new test fixture for that test.

Database fixtures should be wrapped within Traits that encapsulate loading and unloading of the data, as well as providing a connection to said database.

Mysql will have a base trait "MySQLFixture" that, if mixed into a class, will load an empty mysql database and expose a connection to it.
MongoDB will have a base trait "MongoDBFixture" that will perform the equivalent functionality for mongodb.

It will also define the following: 'loadFixture' method 


```scala
  def loadFixture(dataFile: File): Unit
  def withConnection(x: => AnyRef): Unit
```

loadFixture will throw an Exception if the fixture could not be loaded.  If an
Exception is thrown, any connections to the resource will be closed before
thrown.

It's expected that useful datasets can be represented by extending the base fixtures and encapsulating loads of particular datasets.

## Open Questions

* Mocking salesforce with Betamax.  Mocking state changes may not be possible,
  given that no "real" *new* creates or updates are actually happening.

* Mocking the document manager -- This may be an intrinsic limitation and not
  easily mockable.

* Rules around allowed test access to its fixture data.  ie: how can a test
  safely reload its fixture if required without conflicting with other tests?

* Specifics around loading mongodb fixture data.

* Inbound requests to Arinna from salesforce.  We need mocks that implement 
  inbound requests.

* Semantics around usage of fixture connections remains unclear.  (Specific types of connections handed to the caller block are not yet known.)
