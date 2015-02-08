@page page_tech_info Technical Information


Communication
=============

Even though the Jade framework facilitates to great extents the task of communicating between agents by taking care of low level networking details, it is quite easy to get things wrong. It is designed to support a large set of standards, formats, and other details between many platforms, so things can get confusing. Not to mention that handling this using concurrent tasks and the Jade concept of _behaviours_ can easily lead to unwanted side effects.

This framework offers a basic set of classes intended to simplify reliable communication between agents within the context of SmartDESC simulations.


Message Handlers
----------------

Communication should be done by using one of the available `MessageHandler`s provided by the framework in the `ca.smartdesc.comm` package. These can be seen as services launched in the background of the main tasks performed by the component, such as performing calculations at every time step[^background-handlers]. 

Components register handlers during the initialization step, and can then forget about them. Whenever a message of a registered type needs to be processed, its associated handler will take care of it transparently and the result will be available for the main task at the beginning of the next time step.

Message handlers should be used in pairs, one of each complementary type on two or more components interested in communicating together.

[^background-handlers]: Technically, the message handlers do not really operate in the background of main tasks. They execute their operations whenever other tasks are idle. So be aware that a very long to execute or hanging main task will prevent message handlers to do their job and might lead to other agents hanging or slowing down.


### One-way message handlers ###

The simplest types are one-way handlers: `MessageSender` and `MessageReceiver`. They each have a single method of interest: `send(Message)` and `processMessage(Message)` respectively.

Despite their label, they can certainly be used for bi-directional "conversations" between agents. The only thing is that each communication event processed by these handlers is complete in itself and doesn't require anything from the other agent involved. Conversations are handled on a step-by-step basis, and managed by the main cyclic task of the components using them.


### Two-way message handlers ###

The other type of communication is about requests. In this case, each event is initiated by a `RequestSender`, and handled by a `RequestExecutor` who sends its processed result automatically to the sender without intervention by the executor's main task. The `send(Message)`  method of the `RequestSender` does the same as that of any `MessageSender`, but also activates a listener for the result returned by the `processMessage(Message)` method on the receiving agent having a proper `RequestExecutor` instance.

Depending on the communication delay and progression of time, a complete two-way transaction can happen within a single time step. An agent can send a request at one execution step, and expect to deal with the response as soon as the next step.


Native Jade Communication
-------------------------

Every agent automatically has a `SuperMessageHandler` activated whenever a receiving `MessageHandler` instance is created. The `SuperMessageHandler` is responsible for filtering incoming messages and acting as a garbage collector. It will get rid of messages rejected by all registered handlers, so that the inbox doesn't end up stacking "spam" that would otherwise slow down normal communications. **Note: To be implemented soon**

If, for any reason, you need to communicate through other channels than what is supported by the `MessageHandler`s, you are free to use anything the Jade framework has to offer. `MessageHandler`s should play nicely with native behaviours. However, you will be responsible for ensuring that you can handle any situation by yourself to avoid blocking the message queue or hanging concurrent tasks.


Simulation Time
===============

In the simulation context, time is generally an absolute value representing the number of seconds elapsed since January 1st, 1970. This definition is consistent with UNIX systems and can identify without ambiguity any instant from 1970 to later than 2100 with a precision of one second. By convention in the SmartDESC simulator, this is referred to as the absolute simulation time.

Using an absolute time that can be directly converted to a date allows for the simulator to easily use actual data sourced from a database representing real measurement data.


Global parameters
-----------------

The beginning of a simulation is set using the `start` parameter, which is expressed in absolute time as outlined above. Note that a start time of zero does not necessarily mean that things should start in 1970. It means that the simulation should start at the latest time that already exists on disc for a given configuration. In other words, it will automatically resume where the previous simulation stopped. Only if no suitable data can be found will the simulation start at an absolute time of zero. **Note: this feature is not fully implemented now**.

The end of a simulation is set by one of the `stop` or `duration` parameters, or a combination of both. To define the end in terms of absolute time, set `stop` to the desired value and `duration` to zero. If all that you care for is how long the simulation will last, set `duration` to the desired value and `stop` to zero. Setting both values to zero means that the simulation will run until killed externally. Setting both to non-zero values is illegal and will throw a `ValidityException`.

The timing resolution is defined by the `step` parameter. All agents define their main execution tasks in a `runStep()` method that is performed once per time step. Note that, because of the nature of agent based concurrent processing, the actual timing resolution of events that can be handled is not entirely dictated by this parameter. A brief discussion on this topic will later explain why using a real world example.


Concurrency
-----------

For scalability reasons, there is no such thing as a global simulation clock on the system. By design, a minimum number of resources are shared, hence maximizing thread independence. Each agent must then handle his own progression of time independently. Nevertheless, agents being aware of how they should handle timing and when to proceed to a given time step is crucial for any simulation to run properly. This is enabled by using one of several timing modes and an optional synchronization mechanism.


### Synchronization ###

Technically speaking, the agents sit on top of a custom fast synchronization signalling network. Parents provide signalling objects that each of their children hook up to, and use to handle flags or messages that trickle down/up across the tree _without_ the performance cost related to the overhead of normal inter-agent communication

A synchronization parameter `sync` defines at what simulation time interval this synchronization process should be performed between agents. When enabled, agents will process the time step following a synchronization step only after every agent has confirmed that it is ready to do so. When using this feature, every agent is considered to be part of a synchronized clock domain defined by its position in the configuration tree. 

A word of caution should be added: all agents of the same clock domain should have the same `sync` setting, otherwise the system could hang. Indeed, at some step, some agents will wait for a synchronization signal that others will not send until some other future time step, resulting in deadlock.

In the current implementation, only local synchronization is supported (no synchronization across hosts) and is managed by the `RootContainer`. So every agent is part of the same clock domain, and hence should have the same `sync` parameter value.


### Timing modes ###


#### Free running ###

In the simplest case, the synchronization mechanism described above is not used, and every agent is free to process any time step whenever it is ready to do so. This is referred to as the _free running_ mode.  


#### Fixed steps ###

In this case, there is a direct and constant relationship between simulation time progression and the actual real time elapsed between steps. This is referred to as _fixed steps_, for which an acceleration factor is defined. It basically tells how much faster time progresses in the simulation context compared to real time. For example, an acceleration factor value of 60 would mean that for every second that you wait during a simulation, the equivalent of one minute is being processed.

Knowing the acceleration factor, time will progress evenly on every agent. However, the exact instant at which an agent will start computing a given time step will be slightly offset from any other agent who started running with a certain delay. Moreover, there is a risk that an agent performing significantly time consuming tasks at a given step end up lagging behind others. In this situation, a warning is issued, but simulation goes on.


#### Adaptive steps ###

In _adaptive steps_ mode, synchronization is performed at every step. Every agent then signals to others that he is ready to go further, and waits for others to do the same. In this mode, there is no predictable relationship between actual and simulation times. The time required to complete one simulation time step will be as small as possible, or as long as required.

This practically solves both issues mentioned above, but involves going through a synchronization handshaking mechanism that might be time consuming for very large networks, depending on their structure. Moreover, it prevents interacting with time based agents or external events.


#### Synchronous steps ####

Combining the behaviour of fixed and adaptive modes lead to _synchronous steps_ mode. In this case, simulation steps have a predefined duration, but the synchronization mechanism is always performed before processing the next step. 

When things run smoothly, synchronous and fixed modes are very similar. However, synchronous mode:

-	eliminates most of the offset inherent to fixed steps mode;
-	ensures that no agent will lag behind others, since all agent will always wait for the slowest to complete its task;
-	introduces a synchronization delay between every steps that might be noticeable for large networks;


#### Mixed steps ###

Depending on the situation, it might be useful to use _mixed steps_. In this mode, time progresses evenly and independently on every agent according to the acceleration factor for a given simulation duration, then a synchronization operation is performed before staring a new cycle.

A typical scenario where this mode can be useful is when agents should run quickly and independently, but need to perform slow I/O operations at regular intervals, for example storing their state or outputs to disc. Mixed mode would allow using a large acceleration factor or working in free running mode, while ensuring that I/O operations are completed before going further.


### Configuration ###


Two timing parameters, when combined, define the timing mode in use:

-	`sync`: the period (seconds in simulation time) between steps where the agents should go through the synchronization mechanism;
-	`accel`: the acceleration factor.

A `sync` value of zero means that synchronization should never be performed. An `accel` value of zero means that agents run independently of the actual real time.

Putting this information together, and referring to a time step equal to one, we can summarize with the following table:

| Sync	 	| Accel	| Timing mode
| :----:	| :-----:	| :-------------
| 0	 	| 0	| Free running
| 0 	 	| > 0 	| Fixed steps
| 1 	 	| 0 	| Adaptive steps
| 1 	 	| > 0	| Synchronous steps
| > 1 	 	| 0	| Mixed, free running between sync
| > 1 	 	| > 0 	| Mixed, fixed between sync

Actually, this table conveys the logic for setting parameters. But since the `sync` period is expressed in terms of seconds rather than number of steps, the `sync` thresholds in the table should be expressed relatively to the time step parameter `step`. To be accurate, the values `{0, 1, >1}` in the table should be `{0, 0 < sync <= step, sync > step}`.



Persistence
-----------

### Before and After a Simulation ###

Every agent has the ability to save its current state to disc. At the beginning of every simulation, al agents will do one of two things:

-	if data that correspond to the current configuration exist on disc, the agent will load and use it;
-	otherwise, it will go through initialization using the current configuration, then save the result to disc.

Following this, simulation can run until the end, at which point every agent will save its final state to the same location so that a subsequent simulation can resume from there. 


### During a Simulation ###

Two timing parameters define when an agent should store its current state during the course of simulation. First, at the end of every interval equal to the `save` parameter, agents will store their current state before continuing. 

Setting a `save` parameter larger than the time `step` makes sense if you want the progression of the simulation to depend on events with a high resolution, while only a lower resolution is required for eventual examination and analysis. Also, while the `step` should be consistent between agents, each is free to save its outputs independently. Setting `save` to zero will never save the states during a simulation.

Saving states is relatively fast, as data will remains within the volatile memory of the agent. These saved states will only be written to disc or a database at regular intervals defined by the `persist` parameter. Alternatively, setting `persist` to zero will avoid such I/O access during execution.

Each setting will have an impact on how the simulation progresses and what you get at the end. For very large networks, and depending on your application, you might want to optimize these settings for a proper balance between output resolution, execution speed, and memory usage. The following guidelines should be used:

-	for higher output resolution, choose a small `save` period;
-	for higher speed, choose a large `persist` period;
-	for lower memory usage, choose `persist = save`.

Naturally, setting `persist` < `save` makes no sense.

The consequences that you have to deal with when tweaking these parameters are:

-	high resolution at high speed costs memory;
-	high resolution with limited memory is slow;
-	you sacrifice output resolution to run fast with little memory.

As a final note, you should know that "saving the final state", as referred to in the previous section, actually means that all saved states still in volatile memory will be written to persistent storage at the end of simulation. Hence, if you really care for speed and have plenty of memory available, your best option is to set `persist` to zero, and set `save` as small as you need it to be.

[Back to Index](index.html) / [Developer Tutorial](page_dev_tutorial.html)

