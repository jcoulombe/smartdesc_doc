@page page_get_started Before You Start


What you need
-------------

The software is only intended to run on UNIX systems and its variants. It has been tested on Oracle Linux Server, as well as Mac OS X. To use it in its current form, you need:

-	[Git] for downloading the software;
-	[Maven] for managing all of its dependencies;
-	a [Java Developer Kit (JDK)], version 1.7 or above;
-	[Eclipse] for development;
-	a [Maven plugin], making things much smoother;
-	a [Git plugin], so you can perform code updates from Eclipse if you want;

[Git]: http://git-scm.com
[Maven]: http://maven.apache.org
[Java Developer Kit (JDK)]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
[Eclipse]: http://www.eclipse.org/home/index.php
[Maven plugin]: http://www.eclipse.org/m2e/
[Git plugin]: http://www.eclipse.org/egit/


What you need to know
---------------------

### Fundamental concepts ###


Before going forward with the actual use of the simulator, it might help to read the [overview](@ref page_overview) in order to grasp the general concepts.


### Programming ###

For understanding the technical parts and developing your own components, you need at least some notions of XML and Java. Eventually, the framework will support other languages than Java for building components, but a Java interface/wrapper is, and will remain, required in any case.

Speaking of other languages than Java, please note that the tools required for their seamless and automated integration with other components are not useable yet. If you need to integrate such libraries in the short term, you will have to 1) create a proper interface (JNI, SWIG or other), and 2) manually configure the builder for your local installation. Instructions for doing so will come as soon as possible (unless automated tools come out sooner, whichever comes first).



### Current Status ###

This is not a mature project. You will run into bugs and limitations for sure. 

#### The Good Things

It is converging, and the software can be tamed to behave properly. Moreover, most concepts have now attained a relatively stable level. You can hence start designing your own components based on the information herein, knowing most possibilities and constraints of the framework. 

Just be ready to bring modifications at some point depending on the evolution of the implementation. In particular, expect the following things to change in a (hopefully) not so far future:

-	grouping of contextual data: everything that is related to timing is currently grouped into a `time` set of parameters. However, after reflexion, timing information regarding when to store data should be linked to the `persister` settings, already defining where and how to do so on a per-module basis. As a result, the `time` will only define global settings regarding when a simulation starts, ends, and the timing resolution/synchronization. I think it makes more sense this way. Just keep in mind that the current document relates to how things are _at the moment_, but a future version will change some of this;
-	configuration file format: the preceding point is mostly semantics, and shouldn't have any impact on the implementation of your component classes. However, it will impact the format of configuration files. Moreover, changes will be brought to their structure and format in order to facilitate awareness of configuration modifications (including inter-module dependencies) and data validation. But final decisions and testing have yet to come.

#### The Other Things

The most important things you should be aware of before trying to use the software are that:

-	the configurator needs to be updated and keeps remnants of an early implementation, so its commands are unreliable for the most part. This can be easily overcome by creating/modifying the configuration files with an external editor before loading all settings at once and going through the following phases. This is the approach used throughout this tutorial;
-	exceptions should be properly handled for the most part, and provide useful information to the user via the console or log files. However, work still needs to be done to deal with exception happening during the simulation, thrown by agents.
-	data is not currently validated prior to being processed. Using malformed or invalid elements will eventually result in exceptions being thrown at unpredictable points in the process. A well behaved implementation, using `XML schema` is on its way.
-	distributed execution on several hosts is not currently supported;
-	some optimization is needed before going forward with massively parallel simulations, even on a single host. **Please limit the number of agents running at once**.
-	the reporting phase is not implemented yet. However, all data is saved as readable characters during the simulation phase, so it is easy to look and interpret the results to see if things are running as expected;
-	there is no way for filtering out what a component should save. Currently, a _save_ operation saves the whole state of a component, including data you might not care for;
-	profiling tools for optimizing the performance of components are in the pipeline, but not there yet;


### Please Help ###

If you spot errors in the documentation, if you encounter inconsistent behaviour or bugs using your own models, or if you think something important is missing, please report it. It might just be a transient fact due to the current status of the software, but it might also be something that would otherwise go under the radar. Everyone might benefit from your action.


[Back to Index](index.html) / [Installing the Software](page_install.html)