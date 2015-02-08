@page page_dev_tutorial Developer Tutorial

Digging into existing example code might be enough for you to create your own modules, but here are some additional details that might be helpful.


Execution Steps
---------------


### Through Phases ###

There is not much to say that concern specific modules about the configuration, generation, and report phases. The framework is designed to take care of it by itself. The only thing to care about is that a file defining default values or expressions has to exist (see below), and will be used by the generator.


### During a Simulation ###

During the simulation phase, a controller is instantiated, and a container is created on each host involved. Then agents are created with their specific configuration as parameter. From there, every agent goes through the following steps:

1.	setting contextual data, initializing shared notifiers for synchronization and status messaging, and instantiating a component according to the configuration;
2.	instantiating message handlers;
3.	setting internal component values based on supplied configuration or state;
4.	waiting for other agents to be created and ready to initialize;
5.	executing a one-shot `initialize()` + `resume()` operation, setting internal component values according to the loaded data and other agents;
6.	executing a one-shot `start()` operation, synchronously with other agents;
7.	executing tasks defined in a `runStep()` method periodically, at every time step;
8.	meanwhile, handling communications and synchronizing with other agents, saving states, etc. at rates specified in the configuration;
9.	saving to persistent storage the final state when the last step has occurred;
10.	shutting down after ensuring that all agents are ready to do so.


Base Classes
------------

Basically, all operations are taken care of by two base classes working hand-in-hand: `SDAgent` and `SDComponent`. All agents are the exact same `SDAgent`, and own a `SDComponent` of a specific sub-class that defines its unique behaviour. As a module developer, you have to define what your specific component (a sub-class of `SDComponent`) will do at some of the above steps, and with what specific data.

Note that the `SDComponent` is _not_ an abstract class, so there is a default harmless method defined for each and every step. You only need to override methods where you want your specific component to do something in particular.

The `SDAgent` takes entirely care of steps 1, 3, 4, 8, 9 and 10. It also takes care of _when_ other tasks are executed. The component defines _what_, and _how_ these are done. Specifically, the component must define:

-	the message handlers' types and methods, if any;
-	what constitutes the different sets of data;
-	how to initialize internal values not part of the configuration data, if any. Note that at this point all agents have been created and message handlers are instantiated, so inter-agent communication can be used if required;
-	what to do when the simulation starts;
-	what to do at every time step.



Data
----

The `SDAgent` has variables and shares their references with the `SDComponent`, which defines its own set of data according to his specific tasks.

In addition, an `agent` variable is provided, which is a reference to the `SDAgent` owning the component. It is hence possible for the `SDComponent` to call, using this variable, any method offered by the underlying Jade `Agent` class.


### Metadata ###

Some metadata can be used by all `SDComponent` methods, including:

-	`id`: the unique identifier of the agent;
-	`index`: the index of the `SDAgent`, differentiating it from its siblings;
-	`version`: the current version;
-	`configId`: the unique identifier for the current configuration;


### Contextual Data ###

Contextual data is also accessible by both the `SDAgent` and its `SDComponent`:

-	`time`: the current status and settings related to time, including the convenient method `time.current()` which returns the current simulation time;
-	`parent`: the `id` of the parent agent in the configuration tree;
-	`children`: a list of `id`s for every children in the configuration tree;

A `persister` reference is also part of the contextual data and accessible by both `SDAgent` and `SDComponent`. However, since the base `SDComponent` takes care of all state loading and saving tasks, there should be no reason for classes extending `SDComponent` to use it.


### Module Specific Data ###

Every component can define three sets of data: configuration, state and transient. The transient data complements the state, which is a superset of the configuration. More precisely:

-	configuration data is what the user specifies and that is sufficient to start in a desired state, albeit maybe unpredictable;
-	state data is what should be known to get in a perfectly defined state, including what evolves through time and what could be a unique result of the initial configuration;
-	transient data is what is used by the component, but can be generated from any of the above data, and shouldn't be saved to disk.

Suppose that a component is a pseudo-random generator that takes a seed as an argument, and combines it with the current time to generate a number. The seed is supplied by the user, and is sufficient for the component to be initialized, but leads to an unpredictable state. The seed is the configuration data. Knowing the number that was generated using the seed, on the other hand, is the best way to repeat or resume from a specific simulation time step in the future. It is the state. 

The fact that the state is defined as a superset of the configuration is not a fundamental requirement. It is simply a design decision that makes creating new components very straightforward, as we will see.


Methods
-------


To be done...


Basic Examples
--------------


### Simple Hello ###

Let's use the classic "Hello World" example to see how a new component should be created. In the simplest case, where we only want to print to the console at every time step a message configured by the user, the implementation could be:

	public class SimpleHello extends SDComponent {

		@Element String message;

		public void runStep() {
			System.out.format( "%s says: '%s'\n", id, message);
		}
	}

The `message` variable uses an annotation `@Element`, as defined by the _simple-xml_ library (http://simple.sourceforge.net), and makes it part of the configuration data. The `runStep()` method defines what this agent will do at every step.

There is nothing more to do. Our first component design is complete. 

When initializing, the `message` will be automatically loaded according to the user configuration. And when appropriate, the `runStep()` method will be called according to the settings and progression of time. At every simulation time step, we would get something like:

	sim/hello_1 says 'message specified in configuration'


#### Specifying Defaults ####

Actually, we should provide a default value for the `message`. For this, we create a file called `SimpleHello.xml` with the following content:

	<?xml version="1.0" encoding="UTF-8" ?>
	<configuration>
	  <message>Hello World!</message>
	</configuration>


### Dynamic Hello ###

Let's say that a new type of component, based on the existing `SimpleHello`, should keep track of the number of times the message has been sent to the console. We could write a new class like the following:

	public class DynamicHello extends SimpleHello {

		int nbTimes;

		public void initialize() {
			nbTimes = 0;
		}

		public void runStep() {
			super.runStep();
			nbTimes = nbTimes + 1;
		}
	}


The `DynamicHello` component extends the `SimpleHello` type. Therefore, il will have the same configuration data, plus anything that we define here. But because the `nbTimes` variable has no _simple-xml_ annotation, it will not be part of the configuration. Thus a configuration file using a component of the `DynamicHello` type would only have a `message` to set. The `nbTimes`, on the other hand, will be saved as part of the state of the component whenever required by the `time` settings.

The `nbTimes` value needs to be initialized before use. So we write an `initialize()` method that will be called only the first time the component is instantiated with a given configuration. If a simulation resumes from a saved state, this method will not be invoked and the `nbTimes` value will start from where it left.

Finally, the `runStep()` of the `DynamicHello` shadows the `runStep()` of the `SimpleHello`. But the call to `super.runStep()` ensures that the behaviour of this component will follow that of the `SimpleHello` class it extends, even after future modifications to it.


### Loud Hello ###

Now, suppose that our component should play a sound saying "Hello!" or "Bonjour!" according to the configuration set by the user. We could write it as:

	@Root(strict=false)
	public class LoudHello extends SDComponent {

		@Element  URL   soundFileLocation;
		transient Sound soundFile;

		public void resume() {
			soundFile = load(soundFileLocation);
		}

		public void runStep() {
			play(soundFile);
		}
	}	

The URL, annotated with `@Element`, is set through configuration. The `soundFile`, on the other hand, will be set through the `resume()` method that will be invoked on initialization, as well as at the beginning of _every_ simulation, wether the component data was initialized by loading an existing state or from user supplied configuration. The `runStep()` method will be executed as usual. 

Note that the `soundFile` is `transient`. This is important because we don't want it to be part of the persistent state of the component, for two reasons:

-	there is no point storing it, as it can be re-generated (loaded, in this case) from the configuration or state data in the `resume()` step;
-	it references a large object that, should it be part of the state, would be a waste of time and resources to save periodically.


### Summary ###

Let's summarize what was highlighted through the preceding examples. Concerning data:

-	everything that should not be saved is defined as `transient`;
-	everything else is the state;
-	everything that is annotated using syntax defined by the [simple xml] library is the configuration.

[simple xml]: http://simple.sourceforge.net

Concerning methods:

-	the `initialize()` method will be executed the very first time a component is instantiated with a given configuration;
-	the `resume()` method will be executed at the beginning of every simulation;
-	the `runStep()` method will be executed at every time step during a simulation.


Message Handlers
----------------

Now, let's got through an example where components communicate between each other. There are three things required at each end of a communication channel. You need to:

1.	create a variable of the desired `MessageHandler` subclass type;
2.	give it a type of message to handle (anything that is serializable);
3.	write a method defining what it should do when such a message needs to be processed.

Suppose that a new agent should offer random access to the data stored at a specified URL to any other agent requesting it. We could define its component as follows:

	@Root(strict=false)
	public class DataSource extends SDComponent {

		@Element  URL       dataSource;
		transient DataChunk largeDataChunk;

		transient RequestExecutor dataSender;

		public void addMessageHandler() {
			dataSender = new RequestExecutor(agent, Integer.class) {
				public Serializable execute(Serializable address) {
					return largeDataChunk((Integer) address);
				}
			};
		}

		public void resume() {
			largeDataChunk = load(dataSource);
		}
	}


First, the `DataSource` component has a single configuration parameter: the URL. The `DataChunk` is transient, for the same reason as the `soundFile` above.

We define `dataSender`, a transient variable of the type `RequestExecutor`. In an early step during initialization, the `addMessageHandler()` method will be called by the `SDAgent` owning this component. At that time, the `dataSender` is instantiated as a new `RequestExecutor` with the following characteristics:

-	it will be called whenever a message containing an object of the type `Integer` is received by this `agent`;
-	its `execute()` method will return the item taken from the `largeDataChunk` that is located at the address received;

Once created, this `RequestExecutor` will constantly listen for messages that include an `Integer`, call the newly created `execute()` method when appropriate, and send back the returned value to the one who sent the original message.

There is no need to define any `initialize()` or `runStep()` methods. This component will simply wait for requests and asynchronously reply with proper items.

Agents interested in using this service would instantiate a component defined as:

	@Root(strict=false)
	public class DataUser extends SDComponent {

		@Element String   dataSourceId;
		         DataItem data;
		         boolean  newDataAvailable;

		transient RequestSender askForData;

		public void addMessageHandler() {
			askForData = new RequestSender(agent, DataItem.class) {
				public boolean processReply(Serializable receivedData) {
					data = (DataItem) receivedData;
					newDataAvailable = true;
					return true;
				}
			};
		}

		public void runStep() {
			if (newDataAvailable) {
				// Do something useful with the data;
				newDataAvailable = false;
				// If we are interested in data at newDataAddress...
				Integer newDataAddress;
				askForData.send(newDataAddress, dataSourceId);
			}
		}


First, the component has to know the `id` of the dataSource, which is obtained as a configuration parameter. The component also has a `data` field and a boolean flag specifying if fresh data to be processed has been received or not. None is transient.

It has a `RequestSender` field named `askForData()`. This is the complementary type for the `RequestExecutor` found in the `DataSource`. Upon instantiation, the variable is created with the following characteristics:

-	its receiver (remember that sending a request implies receiving a reply) will be called whenever a message containing an object of the type `DataItem` is received by this `agent`;
-	its `processReply()` method will store the `DataItem` received in `data`, and raise the `newDataAvailable` flag. It will always return `true` because this handler will accept any `DataItem`. Should we have other handlers triggered by the same type, each would have to carefully select proper messages to process and return `false` for those rejected.

During the simulation, accessing data from the remote `DataSource` is done in the following way:

-	when required, the main task `runStep()` calls `askForData.send()` with an `Integer` specifying the address of the item of interest, and the `id` of the source;
-	at some point, the value returned by the `DataSource` will be received and processed in the background by the `processReply()` method;
-	at the beginning of the next step, the received data will be available and processed. The flag will then be reset so that another round can start over.

Depending on the processing time and communication delay, there could be new data coming from the remote agent and ready to be processed at every time step. There could also be as many users of the service offered by the `DataSource` as we want.


#### About Message Types ####

The type of message to handle can be any class that you want, as long as it is serializable. You can create as many classes for this purpose as there are kinds of messages you are interested in and have handlers for. Using this approach makes it easy to manage the conditions for each handler. For instance, defining and using an trivial class defined as:

	public class DataChunkAddress implements Serializable {
		public Integer value;
	}

would be a better type to exchange between agents. It would make trivial later on to add a new handler that has to deal with `Integer`s, without having any impact on the `dataSender()` service already implemented.

#### About Handlers Execution ####

Be aware that message handlers are _not_ independent background tasks. Every `Agent` has its own thread, but it has only one. Hence it can only perform one task at a time. Message handling is done as required whenever the agent is not running another task, during otherwise dead time. A request can be sent, processed remotely, returned, and have its reply processed within a single time step, but this cannot all be done _while inside_ the `runStep()` method. Don't send a request entering the method hoping to use the processed reply before leaving, it will not happen. Worse, if you call `Thread.sleep()` with this intent, you will halt the whole agent, as well as everything else if in synchronized timing mode.


Required Files
--------------

It is important to follow the project structure shown below:

	smartdesc/
		src/
			main/
				java/
					ca/smartdesc/modules/
							myPackage/
								MyModule.java
				resources/
					ca/smartdesc/modules/
							myPackage/
								MyModule.xml
								MyModule.xsd
			test/
				java/
					ca/smartdesc/modules/
							myPackage/
								TestMyModule.java
				resources/
					ca/smartdesc/modules/
							myPackage/
								FirstTestInput.xml
								FirstTestExpectedResult.xml
								SecondTest.xml

Descriptions:

-	`main/java/.../myPackage/MyModule.java`: the implementation class of the component;

-	`main/resources/.../myPackage/MyModule.xml`: the default expressions

-	`main/resources/.../myPackage/MyModule.xsd`: the validation schema (**later**)

-	`test/java/.../myPackage/TestMyModule.java`: the unit test class (**later**)

-	`test/resources/.../myPackage/*`: any file useful for tests
-	

Testing Your Work
-----------------

To do...

For now, just run any new module as for the tutorial examples and dig into the log file.


Create Your Own
---------------

Now that you know all of this, you can start creating your own modules for your own purposes.

As a starting point, you should create a new package under `ca.smartdesc.modules` by copying the template `ca.smartdesc.modules.template`. The easiest way to do this is by using the Eclipse interface. From the _package explorer_ view, right-click on the `ca.smartdesc.modules.template` package, then _copy/paste_ it with the name you want. All required labels will be automatically adjusted (package name, class name, logger, etc.).

The template is generously commented so that you can probably manage your get your variant to do what you want quite shortly and easily.

You can also complement this by inspecting how the following demo is implemented.

[Back to Index](index.html) / [Demo](page_demo.html)
