Fast tests, fast builds
==============================

* functional tests -> unit tests
* remove features from the graphical interface
* know your system (HD is not important, RAM & processors are, watch out for quirks such as random on Linux)
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
* run your build in parallel
* upgrade your system
