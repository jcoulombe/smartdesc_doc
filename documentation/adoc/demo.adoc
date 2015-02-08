@page page_demo Demo


**Note: the reporting phase is not implemented yet. However**, a real world example that will serve as a use case for reports already exists, and is presented here.

We will now proceed to a crude simulation of a hockey game. Despite its simplicity and apparent lack of "scienceness", this example is well suited for agent based simulation, and demonstrates some of its most salient features. A short discussion on this topic follows the description of the implementation. And, as most sports, it is a good statistics collecting example.

In this scenario, the action takes place on a rink divided in 9 zones (0-8), with the nets located in zones 1 and 7. Players are agents occupying one specific zone at any time. In the `ca.smartdesc.examples.hockey` package, there are two subclasses of the `Player` type: `Forward` and `Goaltender`. Other classes include a `Referee`, keeping track of the score, and a `Team`, simply grouping players together so that they can be easily referenced in the configuration file. The `Team` is a basic `SDComponent` with no particular behaviour.

Players can take actions (keep, pass, drop, shoot or block the puck, request a pass, check an opponent, or score and celebrate a goal) based on the location and status of the puck (free or handled by a player), their own location and that of other players. All forwards can also skate from the zone they are in to an adjacent one. 

Note that players take their own decisions independently of other players' concurrent decisions.

Actions and moves performed by players are signalled to others using inter-agent messaging handlers. However, the success of their actions depends on the other players simultaneous decisions and, most importantly, the timing of their execution. For example, two players can try to grab a free puck in the zone that they both occupy at a given time step, but only the faster of the two will end up carrying the puck. A player can try to check an opponent handling the puck so that it becomes free, but he will only succeed if he does it before the puck carrier skates to another zone or passes the puck to a teammate.

The game starts by setting the puck free in the centre of the ice (zone 4). Forwards can then start taking actions according to their own location, initially attributed at random, and a set of simple rules. Namely, if a forward is:

- in the same zone as the puck, which is free, he tries to grab it;
- in the same zone as an opponent carrying the puck, he tries a body check in order to make the puck available for a future grab;
- checked by an opponent while carrying the puck, he loses the puck which becomes free in the same zone;
- not carrying the puck, but is in the zone of the opponent goaltender, he requests a pass from the puck holder;
- carrying the puck and receives a pass request, he attempts to pass by dropping the puck in the zone of the requesting player, who will have to grab it before an opponent does;
- carrying the puck in the zone of the opponent goaltender, he shoots to the net, which implies dropping the puck so that in case of a save by the goalie, any player in the zone has a chance of grabbing it afterwards;
- in none of the above situations, he randomly skates to a nearby zone or stays where he is.

There is only one rule followed by a goaltender: if he faces a shot, he tries to block it. He has a probability of succeeding defined by his saving average parameter. In case he fails, he notifies the referee who will update the score.

Players will be notified of a goal scored by a teammate, and then celebrate.

To run a simulation for the equivalent of a period of 20 minutes, just enter the following commands:

	smartdesc configure -l examples/hockey/HockeyPeriod.xml
	smartdesc simulate

Of particular interest is that this scenario takes advantage of some important features of agent-based simulation (other than inherent scalability through distributed execution): the simulation progresses in a _non-deterministic_ way according to the timing of events that happen _asynchronously within sub-timestep intervals_. As a result, fine-grained execution can be simulated, even with a crude time step.

Also note that this is a scenario (and specific implementation) where the use of an acceleration factor is important, rather than progressing at any rate. Otherwise, some communication messages might be handled at time steps where the message is not relevant anymore, so players would not behave according to the accurate current situation on the ice at all time.


TODO: Use the hockey example for continuing the tutorial regarding reports.

Collecting data
---------------


Aggregate data from players saved states;


Formatting the Output
---------------------

Use various output formatters:

-	game summary/statistics as text listings (goals, shots, time of events);
-	player/puck position evolution through time as .csv and matlab formats.


Final Words
===========

For further details, see the implementation and comments in the `ca.smartdesc.examples.hockey` package. Hopefully, you will notice how straightforward it is, using the framework, to create a simulation involving independent decision making models that would otherwise probably be much more contrived. 

Although keep in mind that this implementation has its flaws and should not be followed as a "best practice" example. In fact, this scenario was elaborated for defining and testing agent communication mechanisms required and supported by the framework. A better implementation, and what you should aim for in your own case, could handle communications more wisely and keep their number down. Otherwise, performance might suffer when scaling up, and you might run into concurrency issues.

Good luck!

[Back to Index](index.html)
