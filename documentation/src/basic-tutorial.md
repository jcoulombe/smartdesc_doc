@page page_user_tutorial Basic Usage Tutorial

Hello World
-----------

### Configuration ###

As in most tutorials, let's start with a classic "Hello World" example.

First, create and enter a directory that will be the root for everything from now on, and initialize a new SmartDESC workspace:

	cd path/of/your/choice/tutorial
	smartdesc init

You should notice that a new directory has been created. This is where all automatically generated files can eventually be found. 

Let's start by loading a file distributed with the source code:

	smartdesc configure -l examples/hello/Hello.xml

To see what you have in hands at this point, you can print the current configuration with the `-p` option:

	smartdesc configure -p

What you find is basically 

-	an element `<sim>` 
	-	implemented by the class `ca.smartdesc.core.components.RootContainer`;
	-	configured to run for a duration of `5`. 
-	a child element `hello` implemented by the class `ca.smartdesc.examples.tutorial.HelloWorld`.

Let's see what we get if we just run a simulation as-is, and wait for a few seconds:

	smartdesc simulate

You should have seen, among other things, a few `INFO  Hello World!` messages appearing at a rate of one per second. If not, take a look at the [FAQ](@ref page_faq), you might find a solution.

To understand what just happened, let's look at the files generated inside the workspace. First, take a look at the output of the _generate_ phase by opening `smartdesc.workspace/Generate.xml`

Note that some info has been injected into the original tree for every component, including:

-	metadata:
	-	an `<id>` uniquely identifying the component in the whole tree;
	-	an `<index>`, identifying the component by number, differentiating it from its pairs;
	-	a `<version>` identifier;
	-	a `<configId>` number that can be later used to determine if the configuration has changed or not.

-	contextual settings:
	-	a `<time>` element that will be a topic in itself later;
	-	a `<persister>` which identifies where and how data is to be saved/loaded.
	-	a `<node>` element identifying the component's parent and children, so that the agent is aware of its position in the virtual hierarchy once running independently;

-	component configuration data:
	-	a `<data>` element, which includes all data specific to the related `<class>`.

These are either generated automatically or loaded from default values associated with the selected components. Except, of course, for everything that we specified explicitly which remains untouched.

We are free to change any field in the `<data>` subtree. Let's save a local copy of the current configuration in the current directory for further examination and modifications.

	smartdesc configure -s myHello.xml

Now, copy the following lines

	<data>
      <message>Bonjour!<message>
    </data>

as a child of the `sim/hello/cfg` element in the `myHello.xml` file that was created in your current directory. You should now have something similar to:

       . . .
        <hello>
          <cfg>
            <class>ca.smartdesc.examples.example.HelloWorld</class>
	        <data>
    	      <message>Bonjour!</message>
    	    </data>
          </cfg>
        </hello>
       . . .


Save **with UTF-8 encoding**, and re-run the simulation taking care of loading the new data by entering the next two lines. You should get the obvious result.

	smartdesc configure -l myHello.xml
	smartdesc simulate

This is, as expected, the most basic example. But it shows how you can configure the behaviour of components at runtime with no modification to the underlying code.


### Expressions ###

The point of using a multi-agent system is to run several of them in parallel, right? So let's duplicate our component. In `myHello.xml`, insert `<index>1..5</index>` under `<cfg>`. 

	. . .
	<cfg>
        <class>ca.smartdesc.examples.hello.HelloWorld</class>
        <index>1..5</index>
        . . .

Again, executing

	smartdesc configure -l myHello.xml
	smartdesc simulate

will lead to what you certainly expected. Unfortunately, it is impossible to differentiate components. So we currently have a multi-agent system, but they are all alike and perform the same task the same way.

Instead of writing a different configuration for each of them, let's insert an expression into the message. Insert `${../../index}: ` at the beginning of the message field, which should now look like:

	<message>${../../index}: Bonjour!</message>

The path inside the `${ }` sequence is an XPath expression pointing to a node in the configuration tree, evaluated from the node of the expression itself. In other words, starting from the `message` node, you go up twice to the enclosing `cfg` node, then down to get the value of the `index` node ("`1..5`") that will replace the expression. Re-run the simulation with the usual pair of commands to see the result.

Note that none of the agents actually print "`1..5`" literally. This is because the expression substitution is performed _after_ duplicating the components according to the initial `index` value, hence a unique message is printed by every agent. You can take a look at the `Generate.xml` file inside the workspace folder to see how expressions were translated.

As a side note: you might have also noticed that the sequence is not constant. Since all agents are operating independently in parallel, there is no way to predict which agents will print to the console first. 

One more good thing to know about substitution is that injecting defaults and other automated steps also happen before resolving expressions. It is then possible for expressions to refer to nodes that don't exist in the supplied configuration file, knowing that they will be injected at some point. The only exception is the `configId`, which is a snapshot of the resulting configuration once expressions have been fully resolved and substituted.

Of course, any component can refer to any other component. Also, to facilitate fetching the proper node in more complex trees, a few expressions are available by default and can be nested in other expressions. If you look at the end of the expanded file `Generate.xml`, you will see that entering

	${${//PARENT}/cfg/id}

would be valid for returning the `id` of the parent agent from any node, at any depth inside the configuration sub-tree of a component. For example, you can try this:

	<message>I am '${${//ME}cfg/id}', child of '${${//PARENT}/cfg/id}'.</message>

Oups!

### Troubleshooting ###

Looking at the message that we got, we notice that there is something wrong around a `cfg/id`. Indeed, we should have entered `${${//ME}/cfg/id}`. If you correct this error, you should get the expected result.

Unfortunately, not all errors are so easy to troubleshoot. A good thing to know when you are facing problems is that running 

	smartdesc log

will generally give you much more information about what is going on.

The file that was just printed to the console can also be viewed using any tool that you like by opening `smartdesc.log` inside the `logs` directory inside the workspace.

#### The Workspace

It is a good idea to take a look at other things that you find in the workspace. What can help the most might be to change logging setting in `setup.xml`, to look at the `Generate.xml` file, error dumps, or simulation results in `sim` and its subdirectories.

Finally, the workspace is independent and self contained. Do not hesitate to remove it entirely or any of its items if you want to start on a new base. Also, you can backup or share your work in any state simply by creating a copy of it. You or anyone else should be able to resume from that.

#### Logging ####

You can find in the `setup.xml` file a list of loggers having specific names and levels. The available levels are, in decreasing order of severity: `ERROR`, `WARN`, `INFO`, `DEBUG`, and `TRACE`. Only messages with a severity equal or above `level` will get printed.

If you are using stable components and care for speed, you might benefit significantly from setting a higher threshold. On the other hand, if you are developing your band new component, you might prefer the `DEBUG` or `TRACE` levels. But to avoid being flooded with messages of no interest, you can set per-class or per-package threshold levels. **Details to come…**

Now that the basics concerning the usage of the simulator are covered, let's dig into more developer oriented topics.


[Back to Index](index.html) / [Technical Information](page_tech_info.html)
