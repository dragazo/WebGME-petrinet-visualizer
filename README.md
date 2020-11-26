# WebGME-petrinet-visualizer

`WebGME-petrinet-visualizer` is a an extension for [WebGME](https://github.com/webgme/webgme) that provides a metamodel and tools for constructing, simulating, and doing some analysis of [Petri nets](https://en.wikipedia.org/wiki/Petri_net).

## Definition

Petri nets are a type of bipartite digraph with two categories of vertices: *places* and *transitions*.
A `Place` is conceptually a location where information or some other relevant material can be stored, while a `Transition` is some process that transforms zero or more inputs into zero or more outputs.
Places and Transitions are connected by directed edges (the graph is bipartite with respect to places and transitions).
All Places have an associated integer marking denoting their amount of material.
A transition is *enabled* if all of its inputs (places with an edge leading to the transition - aka *inplaces*) have at least marking 1.
*Firing* a transition consumes 1 item from each input and produces 1 item in each output (places connected by an edge from the transition - aka *outplaces*).

## Use Cases

The execution of a Petri net proceeds by successively firing enabled transitions, but this process may be nondeterministic.
By this design, Petri nets are useful tools to model parallel and network processing, with the markings denoting data being consumed and produced by network nodes and/or communicating programs.

## Installation

Firstly, ensure that you have all the required dependencies for [WebGME](https://github.com/webgme/webgme) itself.
This includes having `mongodb` installed and running.

If all you want is a fresh instance of WebGME with this extension, simply clone the repository, grab all dependencies with `npm install`, and start your server via `node ./app.js` from the repository's root directory.
Upon visiting the website (`localhost:8888` by default), you can create a new project based on the `petri` seed (template) project, which contains the required metamodel and some example networks to demonstrate the visualizer.

If you would instead like to add this extension to an existing instance of WebGME, proceed with the following (ensure your WebGME server is not running):

* Copy `src/visualizers/widgets/petriviz` to your widgets directory and copy `src/visualizers/panels/petriviz` to your panels directory. This will also require updating your `src/visualizers/Visualizers.json` file (see the one in this repository for the required entry). This also requires updating `webgme-setup.json` (see ours for the required entry).
* Add `src/seeds/petri` to your seeds. Other files must also be updated, so it is probably easiest to import the raw `src/seeds/petri/petri.webgmex` file directly by executing `webgme new seed -f PATH petri` from your installation's root directory where `PATH` is the path of the `.webgmex` file. This will create a new seed called `petri`.
* Install [`jointjs`](https://www.jointjs.com/), and [`lodash`](https://lodash.com/), which can be done through `npm` with `npm install jointjs lodash`.

After this is completed, simply start up your WebGME server (e.g. `node ./app.js`) and the installation should be completed.
The new seed `petri` should contain the required metamodel to use the visualizer, and has its own examples with their own use-case documentation.

## Modeling

Once installed, start the server and create a new project with the `petri` seed.
You can view the example networks by double-clicking them, but we will now discuss how to construct models.
From the root, drag and drop a `PetriNet` from the left tools panel into the center.
This creates a new, empty Petri net; double-click to enter it.
From here, your should see `Place` and `Transition` in your tools panel.
These are the relevant types for the majority of your modeling needs; simply drag and drop one onto the center to create a new instance.
You can change the names of a place or transition via the `name` attribute in the bottom-right panel.
Finally, draw edges between places and transitions by clicking and dragging from the source (rather, one of the square anchor points therein) and dropping on the destination (anchor point).

Places additionally have an `initMarking` attribute that denotes how many marks it has when the network is started fresh (initial state).
When viewing a `PetriNet` itself (not its content) in the composition (default) view, the set of places and their initial markings are listed below it for convenience.

## Simulation

So far, you have been using the default WebGME `Composition` visualizer, which is where editing hapens.
For simulation, select the `petriviz` visualizer from the top left panel.
This will enter the visualization mode providd by this extension, which is a nicer graph-based display with colors, animations, and simulation/analysis capabilities.

Firstly, note that the `initMarking` attributes of places are applied when you first load the visualizer, and you can reset to this state at any time by either switching out of and back into the visualizer or by clicking the `Reset Markings` (circle arrow) button on the utilities ribbon near the top of the screen.
Markings are displayed visually as nothing (0), a number of black circles (1-12), or otherwise simply as a number.
If you hover over a `Place` and use the mouse wheel, you can manually adjust the markings of the place.
This is meant to allow for quickly and easily testing multiple starting configurations without having to actually modify the model itself.

Transitions which are enabled *(see the definition above)* are painted blue, while transitions which are not enabled are black.
Note that in the event of a deadlock (i.e. no transition is enabled) all transitions are painted red.
Clicking on any enabled (blue) transition will fire it *(see definition above)*.
Note that the packets (circles) which are in transit during the animation are not considered part of the network until they arrive, so enabled/disabled/deadlock colorings may be different before/during/after the firing.
The system is only truly deadlocked if the colorings remain red after all animations are completed.

## Analysis

The `Classify Network` button in the utilities ribbon near the top of the screen instructs the system to perform some theoretical classifications of the displayed model.
Specifically, it will analyze the following:

* A free-choice petri net is a network where the inplaces of any two distinct transitions are disjoint.
* A state machine is a network where every transition has exactly one inplace and one outplace.
* A marked graph is a network where every place has exactly one in transition and one out transition (similar to a state machine but with places instead of transitions).
* A workflow net is a network with exactly one place `A` having no in transitions, exactly one place `B` having no out transitions, and where, for any Place or Transition `X`, there exists a path `A->X->B` following the (directed) edges of the digraph.

When you click the `Classify Network` button, you will receive several notifications (near the bottom right of the screen), one for each of the above classifications of network.
Each notification will either state that the network is a given type (by just stating the name of the type), or will state that it is not that type and will provide a counterexample.