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
* avoid any I/O with the system (hd, network)
* h2
* Jetty
* Apache VFS, Spring Resource, JBoss VFS, NIO 2 (?)
* SubEtha SMTP
* be on the look out for new, faster frameworks
* be a constant gardener
* do not test everything
* upgrade your system
* builds become slow one test at a time