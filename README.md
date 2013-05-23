Fast tests, fast builds
=======================

When coding, it is vital to keep your build process fast. A slow build is one of the main reasons why developers stop running tests regularly. It distracts from the task at hand and breaks the work flow. Waiting a few minutes for the tests to run often means opening Twitter... and coming back 15 minutes later.

A fast build is an achievable goal. I have worked on a project where members of the development team were all aware of this. Over the course of a particular year, we doubled the number of lines of code, we doubled the number of lines of test... and the build time remained under 5 minutes.

The following will expose the techniques we used. We'll start with the low hanging fruits, and end with the techniques that will probably be the most painful to apply.


Know your system
----------------

Let's start with the obvious. Yes, a more recent computer helps (up to a point). Yes, some operating systems seem to be slightly faster for some tasks. Sometimes, we find differences between compilers under, say, Linux and Mac OS X.

It does seem that faster processors and more RAM makes a difference (again, up to a point). More surprisingly, we found no significant difference when trying out different hard drives. That is, except if your build process is unusually file system-intensive (which I have seen), you will probably not notice much difference when swapping your trusty hard drive for an SSD.

Some OSes have quirks that are worth watching for. For example, when we noticed rather unpredictable build times under Linux, it turned out that the system process that was generating random numbers could be very slow in some circumstances (you can change this behavior, though, see XXXXXXXXXXXXXXXXX).


Run your build in parallel
--------------------------

Maven and, I assume, many other build frameworks, allow for builds to be run in parallel. This is very handy. However, it requires that your tests may be run in parallel themselves. As it happens, it is probably a good idea to have tests that are independent enough to run in parallel, so this is putting the right kind of pressure on your development practices.


Stub some services in integration tests
---------------------------------------

Depending on what is being testing, it might judicious to replace some services in your integration tests with stubs. Good examples include access to external servers.


Replace servers with memory-based implementations
-------------------------------------------------

Some of the slowest pieces of your system are probably servers. Leaving your process to access another one is frequently very costly.

Consider replacing servers with implementations that can be started from your tests. That means a startup time less than one second and a memory-based implementation.

Good examples include HSQLDB or H2 in place of a regular database, and SubEtha SMTP in place of a mail server.

You'll keep one or two tests that do interact with your target database. However, most, if not all, of your tests will work just as well with a memory-based db. As a bonus, you will want to write dialect-agnostic SQL, which will help you switch vendors in the future, should the need arise.


Avoid interacting with the hard drive
-------------------------------------

Manipulating files usually means that the system must create a handle to a file, monitor access to it, and release the handle when done. This is costly and can also behave strangely when things go wrong (Windows is rather bad at releasing file handles when the build crashes).

A good idea is to work as much as possible in a virtual file system, that is one that remains entirely in memory. In most cases, you will be able to create whole (albeit limited) file systems in memory, for each test, for a very low cost.

Many implementations are available on the JVM. The latest versions of NIO 2 in Java 7 (XXXXXX check this XXXXXX) support this. For older versions of the JDK, consider using Apache VFS, Spring Resource, or JBoss VFS.


Be on the outlook out for new, faster tools
-------------------------------------------

Are you using the fastest implementation of this particular API? How do you know?
You need to keep an eye open for new libraries. Spend a few man-days every months to try them out with your project. Most will not meet your needs, but some will, and might make your code a little bit faster.

This is not limited to libraries. Check out HTTP servers (Jetty will be faster than Tomcat in your tests, and might even be enough in production). Check out outlandish things such as Play! Framework.

Stuff to work on:
-----------------

* functional tests -> unit tests
* remove features from the graphical interface
* mock your services in your integration tests
* remove underused features
* do not implement features that seem slow to test
* avoid GUI-based test tools
* avoid integration test frameworks such as Fitnesse and Cucumber
* test HTML pages, AJAX calls instead of testing via the interface
* be a constant gardener
* do not test everything
* upgrade your system
* builds become slow one test at a time