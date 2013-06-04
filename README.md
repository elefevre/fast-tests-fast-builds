Fast tests, fast builds
=======================

When coding, it is vital to keep your build process fast. A slow build is one of the main reasons why developers stop running tests regularly. It distracts from the task at hand and breaks the work flow. Waiting a few minutes for the tests to run often means opening Twitter... and coming back 15 minutes later.

A fast build is an achievable goal. I have worked on a project where members of the development team were all aware of this. Over the course of a particular year, we doubled the number of lines of code, we doubled the number of lines of test... and the build time remained under 5 minutes.

The following will expose the techniques we used. We'll start with the low hanging fruit, and end with the techniques that will probably be the most painful to apply.


Know your system
----------------

Let's start with the obvious. Yes, a more recent computer helps (up to a point). Yes, some operating systems seem to be slightly faster for some tasks. Sometimes, we find differences between compilers under, say, Linux and Mac OS X.

It does seem that faster processors and more RAM makes a difference (again, up to a point). More surprisingly, we found no significant difference when trying out different hard drives. That is, except if your build process is unusually file system-intensive (which I have seen), you will probably not notice much difference when swapping your trusty hard drive for an SSD.

Some OSes have quirks that are worth watching for. For example, when we noticed rather unpredictable build times under Linux, it turned out that the system process that was generating random numbers could be very slow in some circumstances (you can change this behavior, though, see XXXXXXXXXXXXXXXXX).

And of course, if you can upgrade your system, do so. That will save precious seconds many, many times a day, for little effort.


Run your build in parallel
--------------------------

Maven and, I assume, many other build frameworks, allow for builds to be run in parallel. This is very handy. However, it requires that your tests may also be run in parallel. As it happens, it is probably a good idea to have tests that are independent enough to run in parallel, so this is putting the right kind of pressure on your development practices.


Stub some services in integration tests
---------------------------------------

Depending on what is being testing, it might judicious to replace some services in your integration tests with stubs. Good examples include access to external servers.

Mock libraries sometimes help in this, although simply providing a local implementation of those services is sometimes the simplest option.


Replace servers with memory-based implementations
-------------------------------------------------

Some of the slowest pieces of your system are probably servers. Leaving your process to access another one is frequently very costly.

Consider replacing servers with implementations that can be started from your tests. That means a startup time of less than one second and a memory-based implementation.

Good examples include HSQLDB or H2 in place of a regular database, and SubEtha SMTP in place of a mail server.

You'll keep one or two tests that do interact with your target database. However, most, if not all, of your tests will work just as well with a memory-based db. As a bonus, you will want to write dialect-agnostic SQL, which will help you switch vendors in the future, should the need arise.


Start test servers only once
----------------------------

Even fast test servers can take longer to start than you are willing to bear. By carefully designing your tests, it should be possibly to start them only once for your entire test suite, while still automatically ensuring that the context is cleaned up automatically before production code is being exercized.

A common way of doing that is by starting an HTTP server in a pre-test phase in Maven. I'd recommend starting the server from within your tests, which will make it much easier to run them outside Maven.


Avoid interacting with the hard drive
-------------------------------------

Manipulating files usually means that the system must create a handle to a file, monitor access to it, and release the handle when done. This is costly and can also behave strangely when things go wrong (Windows is rather bad at releasing file handles when the build crashes).

A good idea is to work as much as possible in a virtual file system, that is one that remains entirely in memory. In most cases, you will be able to create whole (albeit limited) file systems in memory, for each test, for a very low cost.

Many implementations are available on the JVM. The latest versions of NIO 2 in Java 7 (XXXXXX check this XXXXXX) support this. For older versions of the JDK, consider using Apache VFS, Spring Resource, or JBoss VFS.


Be on the outlook out for new, faster tools
-------------------------------------------

Are you using the fastest implementation of this particular API? How do you know?
You need to keep an eye open for new libraries. Spend a few man-days every months to try them out with your project. Most will not meet your needs, but some will, and might make your code a little bit faster.

This is not limited to libraries. Check out HTTP servers (Jetty will be faster than Tomcat in your tests, and might even be enough in production). Check out outlandish things such as Play! Framework. Is it good enough? Is it faster?


Move behavior away from the graphical interface
-----------------------------------------------

Testing automatically through the UI is difficult, slow, and hard to maintain. When you feel the need to test a feature through the UI, always put significant effort into moving it to a layer below. In last resort, move this behavior to a separate module that can be tested separately from the UI. Nowadays, JavaScript test frameworks are pretty good and plain HTML pages and AJAX calls are easy to test outside the browser.

This will also allow you to mostly bypass UI-based test tools themselves. In truth, your production UI code is not the only slow piece in this chain. UI test frameworks often require that they are run in a cleanly separated environment, often in a combination of various environments (I was once introduced to a company that had to maintain dozens of servers solely for testing their interface in parallel; their build would only run on those servers and would still take an hour or two). Only introduce them after exhausting all other options. And even then, do all you can to remove them.


Avoid integration test frameworks
---------------------------------

Integration test frameworks such as Fitnesse and Cucumber add a significant layer of complexity. That is, before testing your actual code, some time will be spent just starting their environment, inclusing parsing the tests written in their specific idiom.

By all accounts, they are rather fast at this job. The problem is that, multiplied by the number of tests that tend to be written (you don't use them for just a couple of use cases, right?), it adds significant overhead.

The point of those frameworks is that they help in communicating with functional experts. Is this how they are used at your company? Also, consider whether there are alternatives that would give you the same benefits (XXXXXX link to James Shore's article on FIT XXXXX).

Some developers like Fitnesse and Cucumber because they guide them into writing test that clearly expose the business cases that have been implemented. I'd argue that those goals are achievable by basing your functional tests solely on your usual unit test framework. Careful wording of your test methods and clear presentation of the code within the tests can go a long way towards that goal:

	@Test
	public void users_can_log_into_the_system() {
		createUser("john").withPassword("mypassword").inDatabase();

		givenUserHasOpenedPage("index");

		whenEntering(username("john"), password("mypassword"))//
			.andPressingButton("Login");

		thenTheResultingPageShouldContain("Welcome back, John!");
	}

This is essentially what Cucumber leads you to do. The crucial difference is that it is run using JUnit, very quickly. As a bonus, it becomes trivial to run and debug from your usual IDE.

And if you need to show this test to your functional experts, they should be able to understand it at a glance.


Refactor functional tests into unit tests
-----------------------------------------

I've come to realize that functional tests are often the first thing we tend to write when we do not understand the details of what it being tested. User logging fails? Write a functional test. The data retrieved from the database are not the same as that passed originally? Write a functional test.

The problem is that, over time, you end up with many functional tests that have few differences. Look for those differences. They are often things that can easily be moved into unit tests on a lower layer.

Eventually, you should end up with one or two functional tests that check the "happy path". Exceptional cases will be tested mostly with small, fast unit tests.


Remove underused features
-------------------------

Features that are little used become moldy. Nobody requests new things in them, so it is not worth refactoring them and upgrading them to the latest, fanciest libraries.

Still, they are _there_. They make your code harder to comprehend. They make the rest of the code harder to refactor. And they make your build slower.

Spend some energy into checking what features are used the least. Some teams use fancy monitoring systems and gather statistics on what is being used the most. Failing that, simply check your HTTP stats, or what feature has the least requests for bug fixes (a feature that does not have bug requests might be dead -- users who care the most complain a lot).

And if it makes business sense, dump the thing.


Refrain from implementing slow features
---------------------------------------

You will occasionally be requested to implement features that would put a heavy burden on your build system or on your tests. Examples include selecting a framework such as GWT, implementing business logic through the UI, writing exhaustive logs, etc.

Such features should be seen with a critical eye by the technical team. When possible, suggest alternatives. Offer to start with a fast, simple implementation. Otherwise it might be a sign that it should be postponed for a while (in that case, consider paying for options so that you will be more informed when decision time arrives).


Do not test everything (advanced users only)
--------------------------------------------

Sometimes, you need to face reality and accept that not everything will be tested. If enough business knowledge has been moved from the UI to the technical layers, is it still necessary to test the UI? If developers use the UI constantly to verify what they do, is it necessary to test it?

There is, in fact, a case for bypassing tests altogether in some specific situations (see [It’s not about the unit tests](http://agilewarrior.wordpress.com/2012/10/06/its-not-about-the-unit-tests/)). If the part you are testing is simple enough, if the people involved care enough, then tests might have little benefit for you.

Do be very careful of snap judgements, though.


Be a constant gardener
----------------------

In the end, builds become slow one test at a time. The first time you write and run an integration test, it will only add a few seconds to your overall build time. The problem arises once you have many slow tests.

You need to constantly keep an eye open for tests that take longer than others. Not just the very slow ones (you have already tackled those, right?), but the whole category of slow tests. Create a culture where team members always try to move UI-level tests to Integration-level and Integration-level Tests to Unit-level. And keep an eye open for faster alternatives.
