@page page_faq FAQ


Frequently Asked Question
=========================

This is a stub to be enhanced shortly and continuously as _real_ questions arise.


General
-------

### How do I know what version of JVM I am using?

Just enter

	java -version

You should be running a version 1.7 or higher for the software to work. If you know such a version is installed on your computer but the above command tells you that you are running an older version, check your `JAVA_HOME` environment variable and set it accordingly.


### What about a GUI? ###

There is no such thing now, but it would be nice to create and configure networks using libraries, etc. If anyone has time and experience, I would be happy to have some help, at least in tweaking the current version's I/O in order to better support a GUI at a later time.


Using the software
--------------------

### I'm not familiar with _xyz_. Where can I get more info?

provide introduction links for:

-	Jade;
-	XML;
-	XPath;
-	XSLT, Schema, etc.;


Development
-----------


### How can I improve the performance? ###

Hints to come…

### Why is the `src/test/java` folder mostly empty? ###


This situation is temporary, until some revision to its content is done. Most current tests are written for classes that have evolved and haven't been update accordingly. So most fail badly.



### todo ###

provide details about (de)serialization of transient values and `NullPointer`s.


Problems
--------

### Why is the simulator hanging just before launching agents?

There are a few possible causes:

-	a network connection problem. Make sure your network is responding, or run offline by shutting down any port that may be trying to connect unsuccessfully (Wi-Fi behind a firewall, etc.)

-	a `No ICP active` kind of error. There might already be a JVM running already connected to the default port used by Jade (probably due to an earlier crash that didn't close the VM properly). Kill the running VM and restart.

[Back to Index](index.html)