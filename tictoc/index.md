---
layout: page
title: TicToc Tutorial
---

This tutorial to guides you through building and working
with an example simulation model, showing you along the way some
of the commonly used @opp features.

The tutorial is based on the Tictoc example simulation, which you
can find in the <tt>samples/tictoc</tt> directory of your
@opp installation, so you can try out immediately how
the examples work. However, you'll find the tutorial much more useful
if you actually carry out the steps described here.
We assume that you have a good C++ knowledge, and you are in general
familiar with C/C++ development (editing source files, compiling, debugging etc.)
To make the examples easier to follow, all source code in here is
cross-linked to the @opp API documentation.

This document and the TicToc model are an expanded version of
the original TicToc tutorial from Ahmet Sekercioglu (Monash University).

### Contents

  - @ref part1
  - @ref part2
  - @ref part3
  - @ref part4
  - @ref part5
  - @ref part6
  - @ref part7

NEXT: @ref part1
*/

--------------------------------------------------------------------------

## Part 1 - Getting started

@nav{contents,part2}

@tableofcontents

### 1.1 The model

For a start, let us begin with a "network" that consists of two nodes.
The nodes will do something simple: one of the nodes will create a packet,
and the two nodes will keep passing the same packet back and forth.
We'll call the nodes <tt>tic</tt> and <tt>toc</tt>. Later we'll gradually
improve this model, introducing @opp features at each step.

Here are the steps you take to implement your first simulation from scratch.


### 1.2 Setting up the project

Start the @opp IDE by typing <tt>omnetpp</tt> in your terminal. (We assume
that you already have a working @opp installation. If not, please install the latest
version, consulting the <i>Installation Guide</i> as needed.)
Once in the IDE, choose <i>New -> @opp Project</i> from the menu.

<img src="images/newproject.png">

A wizard dialog will appear. Enter <tt>tictoc</tt> as project name,
choose <i>Empty project</i> when asked about the initial content of the project,
then click <i>Finish</i>. An empty project will be created, as you can see
in the <i>Project Explorer</i>.
(Note: Some @opp versions will generate a <tt>package.ned</tt> file into the project.
We don't need it now: delete the file by selecting it and hitting Delete.)

The project will hold all files that belong to our simulation. In our example,
the project consists of a single directory. For larger simulations, the project's
contents are usually sorted into <tt>src/</tt> and <tt>simulations/</tt> folders,
and possibly subfolders underneath them.

@note Using the IDE is entirely optional. Almost all functionality of @opp
(except for some very graphics-intensive and interactive features
like sequence chart browsing and result plotting) is available on
the command line. Model source files can be edited with any text editor,
and @opp provides command-line tools for special tasks such as makefile
creation, message file to C++ translation, result file querying and data export,
and so on.

@note To proceed without the IDE, simply create a directory and create the following
NED, C++ and ini files in it with your favorite text editor.


### 1.3 Adding the NED file

@opp uses NED files to define components and to assemble them into larger units
like networks. We start implementing our model by adding a NED file.
To add the file to the project, right-click the project directory in the
<i>Project Explorer</i> panel on the left, and choose <i>New -> Network Description File (NED)</i>
from the menu. Enter <tt>%tictoc1.ned</tt> when prompted for the file name.

Once created, the file can be edited in the <i>Editor area</i> of the @opp IDE.
The @opp IDE's NED editor has two modes, \e Design and \e Source; one can switch
between them using the tabs at the bottom of the editor. In \e Design mode,
the topology can be edited graphically, using the mouse and the palette on the right.
In \e Source mode, the NED source code can be directly edited as text.
Changes done in one mode will be immediately reflected in the other, so you can
freely switch between modes during editing, and do each change in whichever mode
it is more convenient. (Since NED files are plain text files, you can even use
an external text editor to edit them, although you'll miss syntax highlighting,
content assist, cross-references and other IDE features.)

Switch into \e Source mode, and enter the following:

@dontinclude tictoc1.ned
@skip simple Txc1
@until toc.out
@skipline }

When you're done, switch back to \e Design mode. You should see something like this:

<img src="images/nededitor.png">

The first block in the file declares %Txc1 as a simple module type.
Simple modules are atomic on NED level. They are also active components,
and their behavior is implemented in C++. The declaration also says that
\c Txc1 has an input gate named \c in, and an output gate named \c out.

The second block declares %Tictoc1 as a network. \c %Tictoc1 is assembled from two
submodules, \c tic and \c toc, both instances of the module type \c %Txc1.
\c tic's output gate is connected to \c toc's input gate, and vica versa.
There will be a 100ms propagation delay both ways.

@note You can find a detailed description of the NED language in the
<a href="../manual/index.html#cha:ned-lang" target="blank">@opp Simulation Manual</a>.
(The manual can also be found in the \c doc  directory of your @opp installation.)


### 1.4 Adding the C++ files

We now need to implement the functionality of the Txc1 simple module in C++.
Create a file named \c txc1.cc by choosing <i>New -> Source File</i> from the
project's context menu (or <i>File -> New -> File</i> from the IDE's main menu),
and enter the following content:

@dontinclude txc1.cc
@skip #include
@until // send out the message
@skipline }

The Txc1 simple module type is represented by the C++ class Txc1. The Txc1
class needs to subclass from @opp's cSimpleModule class, and needs to be
registered in @opp with the Define_Module() macro.

@note It is a common mistake to forget the Define_Module() line. If it is missing,
you'll get an error message similar to this one: <i>"Error: Class 'Txc1' not found -- perhaps
its code was not linked in, or the class wasn't registered with Register_Class(), or in
the case of modules and channels, with Define_Module()/Define_Channel()"</i>.

We redefine two methods from cSimpleModule: initialize()
and handleMessage(). They are invoked from the simulation kernel:
the first one only once, and the second one whenever a message arrives at the module.

In initialize() we create a message object (cMessage), and send it out
on gate <tt>out</tt>. Since this gate is connected to the other module's
input gate, the simulation kernel will deliver this message to the other module
in the argument to handleMessage() -- after a 100ms propagation delay
assigned to the link in the NED file. The other module just sends it back
(another 100ms delay), so it will result in a continuous ping-pong.

Messages (packets, frames, jobs, etc) and events (timers, timeouts) are
all represented by cMessage objects (or its subclasses) in @opp.
After you send or schedule them, they will be held by the simulation
kernel in the "scheduled events" or "future events" list until
their time comes and they are delivered to the modules via handleMessage().

Note that there is no stopping condition built into this simulation:
it would continue forever. You will be able to stop it from the GUI.
(You could also specify a simulation time limit or CPU time limit
in the configuration file, but we don't do that in the tutorial.)


### 1.5  Adding omnetpp.ini

To be able to run the simulation, we need to create an %omnetpp.ini file.
%omnetpp.ini tells the simulation program which network you want to simulate
(as NED files may contain several networks), you can pass parameters
to the model, explicitly specify seeds for the random number generators, etc.

Create an %omnetpp.ini file using the <i>File -> New -> Initialization file (INI)</i>
menu item. The new file will open in an <i>Inifile Editor</i>.
As the NED Editor, the Inifile Editor also has two modes, \e Form and \e Source,
which edit the same content. The former is more suitable for configuring the
simulation kernel, and the latter for entering module parameters.

For now, just switch to \e Source mode and enter the following:

@code
[General]
network = Tictoc1
@endcode

You can verify the result in \e Form mode:

<img src="images/inieditor.png" width="650px">

tictoc2 and further steps will all share a common @ref omnetpp.ini file.

We are now done with creating the first model, and ready to compile and run it.

Sources: @ref tictoc1.ned, @ref txc1.cc, @ref omnetpp.ini

@nav{contents,part2}
*/

--------------------------------------------------------------------------

/**
@page part2 Part 2 - Running the simulation

@nav{part1,part3}

@tableofcontents

### 2.1 Launching the simulation program

Once you complete the above steps, you can launch the simulation by selecting
%omnetpp.ini (in either the editor area or the <i>Project Explorer</i>),
and pressing the \e Run button.

<img src="images/run.png">

The IDE will build your project automatically. If there are compilation errors,
you need to rectify those until you get an error-free compilation and linking.
You can manually trigger a build by hitting choosing <i>Project -> Build All</i>
from the menu, or hitting <i>Ctrl+B</i>.

@note If you want to build the simulation executable on the command-line,
create a <i>Makefile</i> using the <tt><b>opp_makemake</b></tt>
command, then enter <tt><b>make</b></tt> to build the project. It will produce
an executable that can be run by entering <tt><b>./tictoc</b></tt>.


### 2.2 Running the simulation

After successfully building and launching your simulation, you should see
a new GUI window appear, similar to the one in the screenshot below.
The window belongs to \e Qtenv, the main @opp simulation runtime GUI.
You should also see the network containing \e tic and \e toc displayed
graphically in the main area.

Press the \e Run button on the toolbar to start the simulation. What you should
see is that \e tic and \e toc are exchanging messages with each other.

<img src="images/tictoc1_3.gif">

The main window toolbar displays the current simulation time. This is virtual time,
it has nothing to do with the actual (or wall-clock) time that the program takes to
execute. Actually, how many seconds you can simulate in one real-world second
depends highly on the speed of your hardware and even more on the nature and
complexity of the simulation model itself.

Note that it takes zero simulation time for a node to process the message.
The only thing that makes the simulation time pass in this model is
the propagation delay on the connections.

You can play with slowing down the animation or making it faster with
the slider at the top of the graphics window. You can stop the simulation
by hitting F8 (equivalent to the STOP button on the toolbar), single-step
through it (F4), run it with (F5) or without (F6) animation.
F7 (express mode) completely turns off tracing features for maximum speed.
Note the event/sec and simsec/sec gauges on the status bar of the
main window (only visible when the simulation is running in fast or express mode).

<i>Exercise: Explore the GUI by running the simulation several times. Try
<i>Run</i>, <i>Run Until</i>, <i>Rebuild Network</i>, and other functions.
</i>

You can exit the simulation program by clicking its Close icon or
choosing <i>File -> Exit</i>.


### 2.3 Debugging

The simulation is just a C++ program, and as such, it often needs to be
debugged while it is being developed. In this section we'll look at the
basics of debugging to help you acquire this vital task.

The simulation can be started in debug mode by clicking the \e Debug
button on the IDE's main toolbar.

<img src="images/debug.png">

This will cause the simulation program to be launched under a debugger
(usually \e gdb). The IDE will also switch into "Debug perspective",
i.e. rearrange its various panes and views to a layout which is better
suited to debugging. You can end the debugging session with the
\e Terminate button (a red square) on the toolbar.


<b>Runtime errors</b>

Debugging is most often needed to track down runtime errors. Let's try it!
First, deliberately introduce an error into the program. In \ref txc1.cc,
duplicate the \c send() line inside \c handleMessage(), so that the code
looks like this:

@code
void Txc1::handleMessage(cMessage *msg)
{
    //...
    send(msg, "out"); // send out the message
    send(msg, "out"); // THIS SHOULD CAUSE AN ERROR
}
@endcode

When you launch the simulation in normal mode (\e Run button) and try to run it,
you'll get an error message like this:

<img src="images/error.png" width="450px">

Now, run the simulation in \e Debug mode. Due to a <i>debug-on-errors</i> option
being enabled by default, the simulation program will stop in the debugger.
You can locate the error by examining the stack trace (the list of nested
function calls) in the \e Debug view:

<img src="images/stacktrace.png" width="600px">

You can see that it was @opp's \e breakIntoDebuggerIfRequested() method that
activated the debugger. From then on, you need to search for a function that
looks familiar, i.e. for one that is part of the model. In our case, that is
the "Txc1::handleMessage() at txc1.cc:54" line. Selecting that line will
show you the corresponding source code in the editor area, and lets you
examine the values of variables in the \e Variables view. This information
will help you determine the cause of the error and fix it.

<b>Crashes</b>

Tracking down crashes i.e. segfaults is similar, let's try that as well.
Undo the previous source code edit (remove the duplicate \c send() line),
and introduce another error. Let's pretend we forgot to create the message
before sending it, and change the following lines in \c initialize()

@code
        cMessage *msg = new cMessage("tictocMsg");
        send(msg, "out");
@endcode

to simply

@code
        cMessage *msg; // no initialization!
        send(msg, "out");
@endcode

When you run the simulation, it will crash. (You will get an error message
similar to "Simulation terminated with exit code: 139"). If you launch the simulation
again, this time in \e Debug mode, the crash will bring you into the debugger.
Once there, you'll be able to locate the error in the \e Debug view and examine
variables, which will help you identify and fix the bug.

<b>Breakpoints</b>

You can also manually place breakpoints into the code. Breakpoints will stop
execution, and let you examine variables, execute the code line-by-line,
or resume execution (until the next breakpint).

A breakpoint can be placed at a specific line in the source code by double-clicking
on the left gutter in the editor, or choosing <i>Toggle Breakpoint</i> from
the context menu. The list of active (and inactive) breakpoints can be examined
in the \e Breakpoints view.

<i>Exercise: Experiment with breakpoints! Place a breakpoint at the beginning of
the \c handleMessage() method function, and run the simulation. Use appropriate
buttons on the toolbar to single-step, continue execution until next time the
breakpoint is hit, and so on.</i>

<b>"Debug next event"</b>

If you did the previous exercise, you must have noticed that the breakpoint
was triggered at each and every event in the Txc1 simple module. In real life
it often occurs that an error only surfaces at, say, the 357th event in that module,
so ideally that's when you'd want to start debugging. It is not very convenient
to have to hit \e Resume 356 times just to get to the place of the error.
A possible solution is to add a \e condition or an \e ignore-count to the
breakpoint (see <i>Breakpoint Properties</i> in its context menu). However,
there is a potentially more convenient solution.

In \e Qtenv, use <i>Run Until</i> to get to the event to be debugged. Then,
choose <i>Simulation -> Debug Next Event</i> from the menu. This will trigger
a breakpoint in the debugger at the beginning of \c handleMessage() of the
next event, and you can start debugging that event.

<img src="images/debugnextevent.png">

### 2.4 The Debug/Run dialog

Let us return to launching simulations once more.

When you launch the simulation program with the \e Run or \e Debug
buttons on the IDE toolbar, settings associated with the launch
are saved in a <i>launch configuration</i>. Launch configurations
can be viewed in the <i>Run/Debug Configurations</i> dialog which
can be opened e.g. by clicking the little \e down arrow next to the
\e Run (\e Debug) toolbar button to open a menu, and choosing
<i>Run (Debug) Configurations...</i> in it. In the same menu, you can also
click the name of a launch configuration (e.g. \e tictoc) while
holding down the Ctrl key to open the dialog with the corresponding
configuration.

The dialog allows you activate various settings for the launch.

<img src="images/launchdialog.png" width="550px">


### 2.5 Visualizing on a Sequence Chart

The @opp simulation kernel can record the message exchanges during the
simulation into an <i>event log file</i>. To enable recording the event log,
check the <i>Record eventlog</i> checkbox in the launch configuration dialog.
Alternatively, you can specify <i>record-eventlog = true</i> in omnetpp.ini,
or even, use the \e Record button in the Qtenv graphical runtime environment
after launching,

The log file can be analyzed later with the <i>Sequence Chart</i> tool in the IDE.
The <tt>results</tt> directory in the project folder contains the <tt>.elog</tt> file.
Double-clicking on it in the @opp IDE opens the Sequence Chart tool,
and the event log tab at the bottom of the window.

@note The resulting log file can be quite large, so enable this feature only
if you really need it.

The following figure has been created with the Sequence Chart tool, and shows
how the message is routed between the different nodes in the network.
In this instance the chart is very simple, but when you have a complex model,
sequence charts can be very valuable in debugging, exploring or documenting
the model's behaviour.

<img src="images/eventlog.png">


Sources: @ref tictoc1.ned, @ref txc1.cc, @ref omnetpp.ini

@nav{part1,part3}
*/

--------------------------------------------------------------------------

/**
@page part3 Part 3 - Enhancing the 2-node TicToc

@nav{part2,part4}

@tableofcontents

### 3.1 Adding icons

Here we make the model look a bit prettier in the GUI. We assign
the "block/routing" icon (the file <tt>images/block/routing.png</tt>), and paint it cyan for <tt>tic</tt>
and yellow for <tt>toc</tt>. This is achieved by adding display strings to the
NED file. The <tt>i=</tt> tag in the display string specifies the icon.

@dontinclude tictoc2.ned
@skip Here we make
@until toc.out
@skipline }

You can see the result here:

<img src="images/step2a.png">


### 3.2 Adding logging

We also modify the C++ code. We add log statements to \e Txc1 so that it
prints what it is doing. @opp provides a sophisticated logging facility
with log levels, log channels, filtering, etc. that are useful for large
and complex models, but in this model we'll use its simplest form
\c EV:

@dontinclude txc2.cc
@skipline EV <<

and

@skipline EV <<

When you run the simulation in the @opp runtime environment, the following output
will appear in the log window:

<img src="images/step2b.png">

You can also open separate output windows for \e tic and \e toc by right-clicking
on their icons and choosing <i>Component log</i> from the menu. This feature
will be useful when you have a large model ("fast scrolling logs syndrome")
and you're interested only in the log messages of specific module.

<img src="images/step2c.png">

Sources: @ref tictoc2.ned, @ref txc2.cc, @ref omnetpp.ini


### 3.3 Adding state variables

In this step we add a counter to the module, and delete the message
after ten exchanges.

We add the counter as a class member:

@dontinclude txc3.cc
@skip class Txc3
@until protected:

We set the variable to 10 in initialize() and decrement in handleMessage(),
that is, on every message arrival. After it reaches zero, the simulation
will run out of events and terminate.

Note the

@dontinclude txc3.cc
@skipline WATCH(c

line in the source: this makes it possible to see the counter value
in the graphical runtime environment.

If you click on <tt>tic</tt>'s icon, the inspector window in the bottom left corner of the main window will display
details about <tt>tic</tt>. Make sure that Children Mode is selected from the toolbar at the top.
The inspector now displays the counter variable.

<img src="images/inspector.png">

As you continue running the simulation, you can follow as the counter
keeps decrementing until it reaches zero.

Sources: @ref tictoc3.ned, @ref txc3.cc, @ref omnetpp.ini


### 3.4 Adding parameters

In this step you'll learn how to add input parameters to the simulation:
we'll turn the "magic number" 10 into a parameter and add a boolean parameter
to decide whether the module should send out the first message in its
initialization code (whether this is a <tt>tic</tt> or a <tt>toc</tt> module).

Module parameters have to be declared in the NED file. The data type can
be numeric, string, bool, or xml (the latter is for easy access to
XML config files), among others.

@dontinclude tictoc4.ned
@skip simple
@until gates

We also have to modify the C++ code to read the parameter in
initialize(), and assign it to the counter.

@dontinclude txc4.cc
@skipline par("limit")

We can use the second parameter to decide whether to send initial message:

@dontinclude txc4.cc
    @skipline par("sendMsgOnInit")

Now, we can assign the parameters in the NED file or from omnetpp.ini.
Assignments in the NED file take precedence. You can define default
values for parameters if you use the <tt>default(...)</tt> syntax
in the NED file. In this case you can either set the value of the
parameter in omnetpp.ini or use the default value provided by the NED file.

Here, we assign one parameter in the NED file:

@dontinclude tictoc4.ned
@skip network
@until connections

and the other in omnetpp.ini:

@dontinclude omnetpp.ini
@skipline Tictoc4.toc

Note that because omnetpp.ini supports wildcards, and parameters
assigned from NED files take precedence over the ones in omnetpp.ini,
we could have used

@code
Tictoc4.t*c.limit=5
@endcode

or

@code
Tictoc4.*.limit=5
@endcode

or even

@verbatim
**.limit=5
@endverbatim

with the same effect. (The difference between * and ** is that * will not match
a dot and ** will.)

In the graphical runtime environment, you can inspect module parameters either in the object tree
on the left-hand side of the main window, or in the Parameters page of
the module inspector (information is shown in the bottom left corner of the main window after
clicking on a module).

The module with the smaller limit will delete the message and thereby
conclude the simulation.

Sources: @ref tictoc4.ned, @ref txc4.cc, @ref omnetpp.ini


### 3.5 Using NED inheritance

If we take a closer look at the NED file we will realize that <tt>tic</tt>
and <tt>toc</tt> differs only in their parameter values and their display string.
We can create a new simple module type by inheriting from an other one and specifying
or overriding some of its parameters. In our case we will derive two simple
module types (<tt>Tic</tt> and <tt>Toc</tt>). Later we can use these types when defining
the submodules in the network.

Deriving from an existing simple module is easy. Here is the base module:

@dontinclude tictoc5.ned
@skip simple Txc5
@until }

And here is the derived module. We just simply specify the parameter values and add some
display properties.

@dontinclude tictoc5.ned
@skip simple Tic5
@until }

The <tt>Toc</tt> module looks similar, but with different parameter values.
@dontinclude tictoc5.ned
@skip simple Toc5
@until }

@note The C++ implementation is inherited from the base simple module (<tt>Txc5</tt>).

Once we created the new simple modules, we can use them as submodule types in our network:

@dontinclude tictoc5.ned
@skip network
@until connections

As you can see, the network definition is much shorter and simpler now.
Inheritance allows you to use common types in your network and avoid
redundant definitions and parameter settings.


### 3.6 Modeling processing delay

In the previous models, <tt>tic</tt> and <tt>toc</tt> immediately sent back the
received message. Here we'll add some timing: <tt>tic</tt> and <tt>toc</tt> will hold the
message for 1 simulated second before sending it back. In @opp
such timing is achieved by the module sending a message to itself.
Such messages are called self-messages (but only because of the way they
are used, otherwise they are ordinary message objects).

We added two cMessage * variables, <tt>event</tt> and <tt>tictocMsg</tt>
to the class, to remember the message we use for timing and message whose
processing delay we are simulating.

@dontinclude txc6.cc
@skip class Txc6
@until public:

We "send" the self-messages with the scheduleAt() function, specifying
when it should be delivered back to the module.

@dontinclude txc6.cc
@skip ::handleMessage
@skipline scheduleAt(

In handleMessage() now we have to differentiate whether a new message
has arrived via the input gate or the self-message came back
(timer expired). Here we are using

@dontinclude txc6.cc
@skipline msg ==

but we could have written

@code
    if (msg->isSelfMessage())
@endcode

as well.

We have left out the counter, to keep the source code small.

While running the simulation you will see the following log output:

<img src="images/step6.png">

Sources: @ref tictoc6.ned, @ref txc6.cc, @ref omnetpp.ini


### 3.7 Random numbers and parameters

In this step we'll introduce random numbers. We change the delay from 1s
to a random value which can be set from the NED file or from omnetpp.ini.
Module parameters are able to return random variables; however, to make
use of this feature we have to read the parameter in <tt>handleMessage()</tt>
every time we use it.

@dontinclude txc7.cc
@skip The "delayTime" module parameter
@until scheduleAt(

In addition, we'll "lose" (delete) the packet with a small (hardcoded) probability.

@dontinclude txc7.cc
@skip uniform(
@until }

We'll assign the parameters in omnetpp.ini:

@dontinclude omnetpp.ini
@skipline Tictoc7.
@skipline Tictoc7.

You can try that no matter how many times you re-run the simulation (or
restart it, Simulate|Rebuild network menu item), you'll get exactly the
same results. This is because @opp uses a deterministic algorithm
(by default the Mersenne Twister RNG) to generate random numbers, and
initializes it to the same seed. This is important for reproducible
simulations. You can experiment with different seeds if you add the
following lines to omnetpp.ini:

@code
[General]
seed-0-mt=532569  # or any other 32-bit value
@endcode

From the syntax you have probably guessed that @opp supports
more than one RNGs. That's right, however, all models in this tutorial
use RNG 0.

<i>Exercise: Try other distributions as well.
</i>

Sources: @ref tictoc8.ned, @ref txc7.cc, @ref omnetpp.ini


### 3.8 Timeout, cancelling timers

In order to get one step closer to modelling networking protocols,
let us transform our model into a stop-and-wait simulation.
This time we'll have separate classes for <tt>tic</tt> and <tt>toc</tt>. The basic
scenario is similar to the previous ones: <tt>tic</tt> and <tt>toc</tt> will be tossing a
message to one another. However, <tt>toc</tt> will "lose" the message with some
nonzero probability, and in that case <tt>tic</tt> will have to resend it.

Here's <tt>toc</tt>'s code:

@dontinclude txc8.cc
@skip Toc8::handleMessage(
@until else

Thanks to the bubble() call in the code, <tt>toc</tt>'ll display a callout whenever
it drops the message.

<img src="images/step8.png">

So, <tt>tic</tt> will start a timer whenever it sends the message. When
the timer expires, we'll assume the message was lost and send another
one. If <tt>toc</tt>'s reply arrives, the timer has to be cancelled.
The timer will be (what else?) a self-message.

@dontinclude txc8.cc
@skip Tic8::handleMessage
@skipline scheduleAt(

Cancelling the timer will be done with the cancelEvent() call. Note that
this does not prevent us from being able to reuse the same
timeout message over and over.

@dontinclude txc8.cc
@skip Tic8::handleMessage
@skipline cancelEvent(

You can read Tic's full source in @ref txc8.cc.

Sources: @ref tictoc8.ned, @ref txc8.cc, @ref omnetpp.ini


### 3.9 Retransmitting the same message

In this step we refine the previous model.
There we just created another packet if we needed to
retransmit. This is OK because the packet didn't contain much, but
in real life it's usually more practical to keep a copy of the original
packet so that we can re-send it without the need to build it again.
Keeping a pointer to the sent message - so we can send it again - might seem easier,
but when the message is destroyed at the other node the pointer becomes invalid.

What we do here is keep the original packet and send only copies of it.
We delete the original when <tt>toc</tt>'s acknowledgement arrives.
To make it easier to visually verify the model, we'll include a message
sequence number in the message names.

In order to avoid handleMessage() growing too large, we'll put the
corresponding code into two new functions, generateNewMessage()
and sendCopyOf() and call them from handleMessage().

The functions:

@dontinclude txc9.cc
@skip Tic9::generateNewMessage
@until }

@dontinclude txc9.cc
@skip Tic9::sendCopyOf
@until }

Sources: @ref tictoc9.ned, @ref txc9.cc, @ref omnetpp.ini

@nav{part2,part4}
*/

--------------------------------------------------------------------------

/**
@page part4 Part 4 - Turning it into a real network

@nav{part3,part5}

@tableofcontents

### 4.1 More than two nodes

Now we'll make a big step: create several <tt>tic</tt> modules and connect
them into a network. For now, we'll keep it simple what they do:
one of the nodes generates a message, and the others keep tossing
it around in random directions until it arrives at
a predetermined destination node.

The NED file will need a few changes. First of all, the Txc module will
need to have multiple input and output gates:

@dontinclude tictoc10.ned
@skip simple Txc10
@until }

The [ ] turns the gates into gate vectors. The size of the vector
(the number of gates) will be determined where we use Txc to build
the network.

@skip network Tictoc10
@until tic[5].out
@until }

Here we created 6 modules as a module vector, and connected them.

The resulting topology looks like this:

<img src="images/step10.png">

In this version, tic[0] will generate the message to be sent around.
This is done in initialize(), with the help of the getIndex() function which
returns the index of the module in the vector.

The meat of the code is the forwardMessage() function which we invoke
from handleMessage() whenever a message arrives at the node. It draws
a random gate number, and sends out message on that gate.

@dontinclude txc10.cc
@skip ::forwardMessage
@until }

When the message arrives at tic[3], its handleMessage() will delete the message.

See the full code in @ref txc10.cc.

<i>Exercise: you'll notice that this simple "routing" is not very efficient:
often the packet keeps bouncing between two nodes for a while before it is sent
to a different direction. This can be improved somewhat if nodes don't send
the packet back to the sender. Implement this. Hints: cMessage::getArrivalGate(),
cGate::getIndex(). Note that if the message didn't arrive via a gate but was
a self-message, then getArrivalGate() returns NULL.
</i>

Sources: @ref tictoc10.ned, @ref txc10.cc, @ref omnetpp.ini


### 4.2 Channels and inner type definitions

Our new network definition is getting quite complex and long, especially
the connections section. Let's try to simplify it. The first thing we
notice is that the connections always use the same <tt>delay</tt> parameter.
It is possible to create types for the connections (they are called channels)
similarly to simple modules. We should create a channel type which specifies the
delay parameter and we will use that type for all connections in the network.

@dontinclude tictoc11.ned
@skip network
@until submodules

As you have noticed we have defined the new channel type inside the network definition
by adding a <tt>types</tt> section. This type definition is only visible inside the
network. It is called as a local or inner type. You can use simple modules as inner types
too, if you wish.

@note We have created the channel by specializing the built-in DelayChannel.
(built-in channels can be found inside the <tt>ned</tt> package. Thats why we used
the full type name <tt>ned.DelayChannel</tt>) after the <tt>extends</tt> keyword.

Now let's check how the <tt>connections</tt> section changed.

@dontinclude tictoc11.ned
@skip connections
@until }

As you see we just specify the channel name inside the connection definition.
This allows to easily change the delay parameter for the whole network.

Sources: @ref tictoc11.ned, @ref txc11.cc, @ref omnetpp.ini


### 4.3 Using two-way connections

If we check the <tt>connections</tt> section a little more, we will realize that
each node pair is connected with two connections. One for each direction.
@opp 4 supports two way connections, so let's use them.

First of all, we have to define two-way (or so called <tt>inout</tt>) gates instead of the
separate <tt>input</tt> and <tt>output</tt> gates we used previously.

@dontinclude tictoc12.ned
@skip simple
@until }

The new <tt>connections</tt> section would look like this:

@dontinclude tictoc12.ned
@skip connections:
@until }

We have modified the gate names so we have to make some modifications to the
C++ code.

@dontinclude txc12.cc
@skip Txc12::forwardMessage
@until }

@note The special $i and $o suffix after the gate name allows us to use the
connection's two direction separately.

Sources: @ref tictoc12.ned, @ref txc12.cc, @ref omnetpp.ini


### 4.4 Defining our message class

In this step the destination address is no longer hardcoded tic[3] -- we draw a
random destination, and we'll add the destination address to the message.

The best way is to subclass cMessage and add destination as a data member.
Hand-coding the message class is usually tedious because it contains
a lot of boilerplate code, so we let @opp generate the class for us.
The message class specification is in tictoc13.msg:

@dontinclude tictoc13.msg
@skip message TicTocMsg13
@until }

@note See <a href="../manual/index.html#cha:msg-def" target="_blank">Section 6</a> of the @opp manual for more details on messages.

The makefile is set up so that the message compiler, opp_msgc is invoked
and it generates tictoc13_m.h and tictoc13_m.cc from the message declaration
(The file names are generated from the tictoc13.msg file name, not the message type name).
They will contain a generated TicTocMsg13 class subclassed from cMessage;
the class will have getter and setter methods for every field.

We'll include tictoc13_m.h into our C++ code, and we can use TicTocMsg13 as
any other class.

@dontinclude txc13.cc
@skipline tictoc13_m.h

For example, we use the following lines in generateMessage() to create the
message and fill its fields.

@skip ::generateMessage(
@skip TicTocMsg13 *msg
@until return msg

Then, handleMessage() begins like this:

@dontinclude txc13.cc
@skip ::handleMessage(
@until getDestination

In the argument to handleMessage(), we get the message as a cMessage * pointer.
However, we can only access its fields defined in TicTocMsg13 if we cast
msg to TicTocMsg13 *. Plain C-style cast (<code>(TicTocMsg13 *)msg</code>)
is not safe because if the message is <i>not</i> a TicTocMsg13 after all
the program will just crash, causing an error which is difficult to explore.

C++ offers a solution which is called dynamic_cast. Here we use check_and_cast<>()
which is provided by @opp: it tries to cast the pointer via dynamic_cast,
and if it fails it stops the simulation with an error message, similar to the
following:

<img src="images/step13e.png">

In the next line, we check if the destination address is the same as the
node's address. The <tt>getIndex()</tt> member function returns the index
of the module in the submodule vector (remember, in the NED file we
declarared it as <tt>tic: Txc13[6]</tt>, so our nodes have addresses 0..5).

To make the model execute longer, after a message arrives to its destination
the destination node will generate another message with a random destination
address, and so forth. Read the full code: @ref txc13.cc.

When you run the model, it'll look like this:

<img src="images/step13a.png">

You can click on the messages to see their content in the inspector window.
Double-clicking will open the inspector in a new window.
(You'll either have to temporarily stop the simulation for that,
or to be very fast in handling the mouse). The inspector window
displays lots of useful information; the message fields can be seen
on the Contents page.

<img src="images/step13b.png">

Sources: @ref tictoc13.ned, @ref tictoc13.msg, @ref txc13.cc, @ref omnetpp.ini

<i>Exercise: In this model, there is only one message underway at any
given moment: nodes only generate a message when another message arrives
at them. We did it this way to make it easier to follow the simulation.
Change the module class so that instead, it generates messages periodically.
The interval between messages should be a module parameter, returning
exponentially distributed random numbers.
</i>

@nav{part3,part5}
*/

--------------------------------------------------------------------------

/**
@page part5 Part 5 - Adding statistics collection

@nav{part4,part6}

@tableofcontents

### 5.1 Displaying the number of packets sent/received

To get an overview at runtime how many messages each node sent or
received, we've added two counters to the module class: numSent and numReceived.

@dontinclude txc14.cc
@skip class Txc14
@until protected:

They are set to zero and WATCH'ed in the initialize() method. Now we
can use the Find/inspect objects dialog (Inspect menu; it is also on
the toolbar) to learn how many packets were sent or received by the
various nodes.

<img src="images/step14a.png">

It's true that in this concrete simulation model the numbers will be
roughly the same, so you can only learn from them that intuniform()
works properly. But in real-life simulations it can be very useful that
you can quickly get an overview about the state of various nodes in the
model.

It can be also arranged that this info appears above the module
icons. The <tt>t=</tt> display string tag specifies the text;
we only need to modify the displays string during runtime.
The following code does the job:

@dontinclude txc14.cc
@skip ::refreshDisplay
@until }

And the result looks like this:

<img src="images/step14b.png">

Sources: @ref tictoc14.ned, @ref tictoc14.msg, @ref txc14.cc, @ref omnetpp.ini


### 5.2 Adding statistics collection

The previous simulation model does something interesting enough
so that we can collect some statistics. For example, you may be interested
in the average hop count a message has to travel before reaching
its destination.

We'll record in the hop count of every message upon arrival into
an output vector (a sequence of (time,value) pairs, sort of a time series).
We also calculate mean, standard deviation, minimum, maximum values per node, and
write them into a file at the end of the simulation. Then we'll use
tools from the @opp IDE to analyse the output files.

For that, we add an output vector object (which will record the data into
<tt>Tictoc15-#0.vec</tt>) and a histogram object (which also calculates mean, etc)
to the class.

@dontinclude txc15.cc
@skipline class Txc15
@until protected:

When a message arrives at the destination node, we update the statistics.
The following code has been added to handleMessage():

@skip ::handleMessage
@skipline hopCountVector.record
@skipline hopCountStats.collect

hopCountVector.record() call writes the data into <tt>Tictoc15-#0.vec</tt>.
With a large simulation model or long execution time, the <tt>Tictoc15-#0.vec</tt> file
may grow very large. To handle this situation, you can specifically
disable/enable vector in omnetpp.ini, and you can also specify
a simulation time interval in which you're interested
(data recorded outside this interval will be discarded.)

When you begin a new simulation, the existing <tt>Tictoc15-#0.vec/sca</tt>
files get deleted.

Scalar data (collected by the histogram object in this simulation)
have to be recorded manually, in the finish() function.
finish() gets invoked on successful completion of the simulation,
i.e. not when it's stopped with an error. The recordScalar() calls
in the code below write into the <tt>Tictoc15-#0.sca</tt> file.

@skip ::finish
@until }

The files are stored in the <tt>results/</tt> subdirectory.

You can also view the data during simulation. To do that, right click on a module, and
choose <tt>Open Details</tt>. In the module inspector's Contents page you'll find the hopCountStats
and hopCountVector objects. To open their inspectors, right click on <tt>cLongHistogram hopCountStats</tt> or
<tt>cOutVector HopCount</tt>, and click <tt>Open Graphical View</tt>.

<img src="images/open_details.png">

The inspector:

<img src="images/open_graphical_view.png">

They will be initially empty -- run the simulation in Fast (or even Express) mode to get enough
data to be displayed. After a while you'll get something like this:

<img src="images/step15a.png">

<img src="images/step15b.png">

When you think enough data has been collected, you can stop the simulation
and then we'll analyse the result files (<tt>Tictoc15-#0.vec</tt> and
<tt>Tictoc15-#0.sca</tt>) off-line. You'll need to choose Simulate|Call finish()
from the menu (or click the corresponding toolbar button) before exiting --
this will cause the finish() functions to run and data to be written into
<tt>Tictoc15-#0.sca</tt>.

Sources: @ref tictoc15.ned, @ref tictoc15.msg, @ref txc15.cc, @ref omnetpp.ini


### 5.3 Statistic collection without modifying your model

In the previous step we have added statistic collection to our model.
While we can compute and save any value we wish, usually it is not known
at the time of writing the model, what data the enduser will need.

@opp 4.1 provides an additional mechanism to record values and events.
Any model can emit 'signals' that can carry a value or an object. The
model writer just have to decide what signals to emit, what data to attach
to them and when to emit them. The enduser can attach 'listeners' to these
signals that can process or record these data items. This way the model
code does not have to contain any code that is specific to the statistics
collection and the enduser can freely add additional statistics without
even looking into the C++ code.

We will re-write the statistic collection introduced in the last step to
use signals. First of all, we can safely remove all statistic related variables
from our module. There is no need for the <tt>cOutVector</tt> and
<tt>cLongHistogram</tt> classes either. We will need only a single signal
that carries the <tt>hopCount</tt> of the message at the time of message
arrival at the destination.

First we need to define our signal. The <tt>arrivalSignal</tt> is just an
identifier that can be used later to easily refer to our signal.

@dontinclude txc16.cc
@skipline class Txc16
@until protected:

We must register all signals before using them. The best place to do this
is the <tt>initialize()</tt> method of the module.

@skipline ::initialize()
@until getIndex()

Now we can emit our signal, when the message has arrived to the destination node.

@skipline ::handleMessage(
@until EV <<

As we do not have to save or store anything manually, the <tt>finish()</tt> method
can be deleted. We no longer need it.

The last step is that we have to define the emitted signal also in the NED file.
Declaring signals in the NED file allows you to have all information about your
module in one place. You will see the parameters it takes, its input and output
gates, and also the signals and statistics it provides.

@dontinclude tictoc16.ned
@skipline simple
@until display

Now we can define also a statistic that should be collected by default. Our previous example
has collected statistics (max,min,mean,count etc) about the hop count of the
arriving messages, so let's collect the same data here, too.

The <tt>source</tt> key specifies the signal we want our statistic to attach to.
The <tt>record</tt> key can be used to tell what should be done with the received
data. In our case we sepcify that each value must be saved in a vector file (vector)
and also we need to calculate min,max,mean,count etc. (stats). (NOTE: <tt>stats</tt> is
just a shorthand for min, max, mean, sum, count etc.) With this step we have finished
our model.

Now we have just realized that we would like to see a histogram of the hopCount on the
tic[1] module. On the other hand we are short on disk storage and we are not interested
having the vector data for the first three module <tt>tic</tt> 0,1,2. No problem. We can add our
histogram and remove the unneeded vector recording without even touching the C++ or NED
files. Just open the INI file and modify the statistic recording:

@dontinclude omnetpp.ini
@skipline Tictoc16
@until tic[0..2]

We can configure a wide range of statistics without even looking into the C++ code,
provided that the original model emits the necessary signals for us.

Sources: @ref tictoc16.ned, @ref tictoc16.msg, @ref txc16.cc, @ref omnetpp.ini


### 5.4 Adding figures

@opp can display figures on the canvas, such as text, geometric shapes or images.
These figures can be static, or change dinamically according to what happens in the simulation.
In this case, we will display a static descriptive text, and a dynamic text showing the hop count of the last message that arrived at its destination.

We create figures in @ref tictoc17.ned, with the <tt>\@figure</tt> property.

@dontinclude tictoc17.ned
@skipline Tictoc17
@until hopCount

This creates two text figures named <tt>description</tt> and <tt>lasthopcount</tt>, and sets their positions on the canvas (we place them in the top right corner).
The <tt>font</tt> attribute sets the figure text's font. It has three parameters: <tt>typeface, size, style</tt>. Any one of them
can be omitted to leave the parameter at default. Here we set the description figure's font to bold.

By default the text in <tt>lasthopcount</tt> is static, but we'll
change it when a message arrives. This is done in @ref txc17.cc, in the <tt>handleMessage()</tt> function.

@dontinclude txc17.cc
@skipline hasGUI
@until setText

The figure is represented by the <tt>cTextFigure</tt> C++ class. There are several figure types,
all of them are subclassed from the <a href="file:///home/user/omnetpp-git-tictoc/doc/api/classomnetpp_1_1cFigure.html"><tt>cFigure</tt></a> base class.
We insert the code responsible for updating the figure text after we retreive the <tt>hopcount</tt> variable.

We want to draw the figures on the network's canvas. The <tt>getParentModule()</tt> function returns the parent of the node, ie. the network.
Then the <tt>getCanvas()</tt> function returns the network's canvas, and <tt>getFigure()</tt> gets the figure by name.
Then, we update the figure's text with the <tt>setText()</tt> function.

@note For more information on figures and the canvas, see <a href="../manual/index.html#sec:graphics:canvas" target="_blank">The Canvas</a> section of the @opp manual

When you run the simulation, the figure displays 'last hopCount: N/A' before the arrival of the first message.
Then, it is updated whenever a message arrives at its destination.

<img src="images/step17.png">

@note If the figure text and nodes overlap, press 're-layout'.

@note <img src="images/relayout.png">

In the last few steps, we have collected and displayed statistics. In the next part,
we'll see and analyze them in the IDE.


Sources: @ref tictoc17.ned, @ref tictoc17.msg, @ref txc17.cc, @ref omnetpp.ini

@nav{part4,part6}
*/

--------------------------------------------------------------------------

/**
@page part6 Part 6 - Visualizing the results with the @opp IDE

@nav{part5,part7}

@tableofcontents

### 6.1 Visualizing output scalars and vectors

The @opp IDE can help you to analyze your results. It supports filtering,
processing and displaying vector and scalar data, and can display histograms, too.
The following diagrams have been created with the Result Analysis tool of the IDE.

The <tt>results</tt> directory in the project folder contains .vec and .sca files, which are the files that store the results in vector and scalar form, respectively.
Vectors record data values as a function of time, while scalars typically record aggregate values at the end of the simulation.
To open the Result Analysis tool, double click on either the .vec or the .sca files in the @opp IDE. Both files will be loaded by the Result Analysis tool.
You can find the <tt>Browse Data</tt> tab at the bottom of the Result Analysis tool panel. Here you can browse results by type by switching the various tabs
at the top of the tool panel, ie. Scalars, Vectors, or Histograms. By default, all results of a result type are displayed. You can filter them by the module filter
to view all or some of the individual modules, or the statistic name filter to display different types of statistics, ie. mean, max, min, standard deviation, etc.
You can select some or all of the individual results by highlighting them. If you select multiple results, they will be plotted on one chart. Right click and select Plot to display the figures.

<img src="images/statistics.png">

@note For further information about the charting and processing capabilities,
please refer to the <a href="../UserGuide.pdf" target="_blank"><b>@opp Users Guide</b></a> (you can find it in the <tt>doc</tt> directory of the @opp installation).

Our last model records the <tt>hopCount</tt> of a message each time the message
reaches its destination.
To plot these vectors for all nodes, select the 6 nodes in the browse data tab.
Right click and select Plot.

<img src="images/selectplot2.png">

We can change various options about how the data on the chart is displayed.
Right click on the chart background, and select Properties.
This opens the Edit LineChart window.
In the Lines tab, set <tt>Line type</tt> to Dots, and <tt>Symbol Type</tt> to Dot.

<img src="images/editlinechart2.png" width="850px">

To add a legend to the chart, select <tt>Display legend</tt> on the Legend tab.

<img src="images/displaylegend.png">

The chart looks like the following:

<img src="images/hopcountchart.png">

If we apply a <tt>mean</tt> operation we can see how the <tt>hopCount</tt> in the different
nodes converge to an average.
Right-click the chart, and select <i>Apply -> Mean</i>.
Again, right-click on the chart background, and select \e Properties.
In the \e Lines tab, set <i>Line type</i> to Linear, and <i>Symbol Type</i> to None.
The mean is displayed on the following chart. The lines are easier to see this way because they are thinner.

<img src="images/mean3.png">

Scalar data can be plotted on bar charts.
The next chart displays the mean and the maximum of the \c hopCount of the messages
for each destination node, based on the scalar data recorded at the end of the simulation.
In the Browse data tab, select Scalars. Now select <tt>hop count:max</tt> and <tt>hop count:mean</tt>
for all 6 nodes.


<img src="images/scalars.png">

To create a histogram that shows <tt>hopCount</tt>'s distribution, select Histograms
on the Browse data tab. Select all nodes, and right click | Plot.

<img src="images/histogram.png">

@nav{part5,part7}
*/

--------------------------------------------------------------------------

/**
@nav{part6,closing}

@page part7 Part 7 - Parameter studies

@tableofcontents

### 7.1 The goal

We want to run the simulation with a different number of nodes, and see how
the behavior of the network changes. With @opp you can do parameter studies,
which are multiple simulation runs with different parameter values.

We'll make the number of central nodes (the "handle" in the dumbbell shape) a parameter, and
use the same random routing protocol as before. We're interested in how the average
hop count depends on the number of nodes.

### 7.2 Making the network topology parametric

To parameterize the network, the number of nodes is given as a NED parameter,
<tt>numCentralNodes</tt>. This parameter specifies how many nodes are in the central
section of the network, but doesn't cover the two nodes at each side.

<img src="images/numberofnodes.png">

The total number of nodes including the four nodes on the sides is <tt>numCentralNodes+4</tt>.
The default of the <tt>numCentralNodes</tt> parameter is 2, this corresponds
to the network in the previous step.

@dontinclude tictoc18.ned
@skipline TicToc18
@until Txc18

Now, we must specify that the variable number of nodes should be connected into the dumbbell shape.
First, the two nodes on one side is connected to the third one. Then the the last two nodes on the other side is
connected to the third last. The nodes in the center of the dumbbell can be connected with a for loop.
Starting from the third, each <i>i</i>th node is connected to the <i>i+1</i>th.

@dontinclude tictoc18.ned
@skipline connections
@until numCentralNodes+3

Here is how the network looks like with <tt>numCentralNodes = 4</tt>:

<img src="images/step18.png">

To run the simulation with multiple different values of <tt>numCentralNodes</tt>, we specify
the variable \e N in the ini file:

@dontinclude omnetpp.ini
@skipline numCentralNodes = $

### 7.3 Setting up a parameter study

We specify that \e N should go from 2 to 100, in steps of 2.
This produces about 50 simulation runs. Each can be explored in the graphical user interface, but
simulation batches are often run from the command line interface using the \e Cmdenv runtime environment.

@note You can find more information on variables and parameter studies in the <a href="../manual/index.html#sec:config-sim:parameter-studies" target="_blank">Parameter Studies</a> section of the @opp manual.

To increase the accuracy of the simulation we may need to run the same simulation several times
using different random numbers. These runs are called \e Repetitions and are specified in <tt>omnetpp.ini</tt>:

@dontinclude omnetpp.ini
@skipline repeat = 4

This means that each simulation run will be executed four times, each time with a different seed for the RNGs.
This produces more samples, which can be averaged. With more repetitions, the results will increasingly converge
to the expected values.

### 7.4 Running the parameter study

Now, we can run the simulations. In the dropdown menu of the \e Run icon, select <i>Run Configurations</i>.

<img src="images/runconfig.png">

In the <i>Run Configurations</i> dialog, select the config name, make sure \e Cmdenv is selected as the user interface.

<img src="images/runconfig2.png">

If you have a multicore CPU, you can specify how many simulations to run concurrently.

@note Alternatively, you can run the simulation batches from the command line with \c opp_runall tool
with the following command:

@code
opp_runall -j4 ./tictoc -u Cmdenv -c TicToc18
@endcode

The -j parameter specifies the number of CPU cores, the \c -u parameter the user interface, and \c -c the config to run.


### 7.4 Analyzing the results

Now, we can visualize and analyze the data we've collected from the simulation runs.
We'll display the average hop count for messages that reach their destinations vs \e N, the number of central nodes.
Additionally, we will display the average number of packets that reached their destinations vs \e N.
The analysis file <tt>Tictoc18.anf</tt> contains the dataset we will use for the visualization.

<img src="images/dataset.png">

These two average scalars are not recorded during the simulation, we will have to compute them from the available data.

The hop count is recorded at each node when a message arrives, so the mean of hop count will be available as a statistic.
But this is recorded per node, and we're interested in the average of the mean hop count for all nodes.
The 'Compute Scalar' dataset node can be used to compute scalar statistics from other scalars.
We compute <tt>AvgHopCount</tt> as <tt>mean(**.'hopCount:stats:mean')</tt>.

We're also interested in the average number of packets that arrive at their destination.
The count of the arrived packets is available at each node. We can compute their average,
<tt>AvgNumPackets</tt> as <tt>mean('hopCount:stats:count')</tt>.

@note Refer to the chapter "Using the Analysis Editor" in the User Guide for more information on datasets. You can find it in the '/doc' directory of your
@opp installation.

Then, we plot these two computed scalars against \e N in two scatter charts. The data for different repetitions is automatically averaged.
Here is the average hop count vs \e N:

<img src="images/avghopcount.png">

The average hop count increases as the network gets larger, as packets travel more to reach their destination.
The increase is polynomial. Notice that there are missing values at the far right of the chart.
This is because in such a large network, some packets might not reach their destination in the simulation time limit.
When no packets arrive at a node, the hop count statistic will be \e NaN (not a number) for that node.
When there is a \e NaN in any mathematical expression, its result will be also \e NaN.
Thus it takes just one node in all the simulation runs to have a \e NaN statistic, and the average will be \e NaN, and there'll be no data to display.
This can be remedied by increasing the simulation time limit, so more packets have a chance to arrive.

Below is the average number of packets that arrived vs \e N:

<img src="images/avgnumpackets.png">

Notice that the Y axis is logarithmic. The average number of packets that arrive decreases polynomially
as \e N increases, and the network gets larger.

@nav{part6,closing}
*/

--------------------------------------------------------------------------

/**
@nav{part7,contents}

@page closing Closing words

### Congratulations!

You have successfully completed this tutorial! You have gained a good overview
and the basic skills to work with @opp, from writing simulations to analyzing
results. To go to the next level, we recommend you to read the <i>Simulation Manual</i>
and skim through the <i>User Guide</i>.

Comments and suggestions regarding this tutorial will be very much appreciated.

*/
--------------------------------------------------------------------------


/// @page tictoc1.ned tictoc1.ned
/// @include tictoc1.ned

/// @page tictoc2.ned tictoc2.ned
/// @include tictoc2.ned

/// @page tictoc3.ned tictoc3.ned
/// @include tictoc3.ned

/// @page tictoc4.ned tictoc4.ned
/// @include tictoc4.ned

/// @page tictoc5.ned tictoc5.ned
/// @include tictoc5.ned

/// @page tictoc6.ned tictoc6.ned
/// @include tictoc6.ned

/// @page tictoc7.ned tictoc7.ned
/// @include tictoc7.ned

/// @page tictoc8.ned tictoc8.ned
/// @include tictoc8.ned

/// @page tictoc9.ned tictoc9.ned
/// @include tictoc9.ned

/// @page tictoc10.ned tictoc10.ned
/// @include tictoc10.ned

/// @page tictoc11.ned tictoc11.ned
/// @include tictoc11.ned

/// @page tictoc12.ned tictoc12.ned
/// @include tictoc12.ned

/// @page tictoc13.ned tictoc13.ned
/// @include tictoc13.ned

/// @page tictoc14.ned tictoc14.ned
/// @include tictoc14.ned

/// @page tictoc15.ned tictoc15.ned
/// @include tictoc15.ned

/// @page tictoc16.ned tictoc16.ned
/// @include tictoc16.ned

/// @page tictoc17.ned tictoc17.ned
/// @include tictoc17.ned

/// @page tictoc18.ned tictoc18.ned
/// @include tictoc18.ned

/// @page txc1.cc txc1.cc
/// @include txc1.cc

/// @page txc2.cc txc2.cc
/// @include txc2.cc

/// @page txc3.cc txc3.cc
/// @include txc3.cc

/// @page txc4.cc txc4.cc
/// @include txc4.cc

/// @page txc6.cc txc6.cc
/// @include txc6.cc

/// @page txc7.cc txc7.cc
/// @include txc7.cc

/// @page txc8.cc txc8.cc
/// @include txc8.cc

/// @page txc9.cc txc9.cc
/// @include txc9.cc

/// @page txc10.cc txc10.cc
/// @include txc10.cc

/// @page txc11.cc txc11.cc
/// @include txc11.cc

/// @page txc12.cc txc12.cc
/// @include txc12.cc

/// @page txc13.cc txc13.cc
/// @include txc13.cc

/// @page txc14.cc txc14.cc
/// @include txc14.cc

/// @page txc15.cc txc15.cc
/// @include txc15.cc

/// @page txc16.cc txc16.cc
/// @include txc16.cc

/// @page txc17.cc txc17.cc
/// @include txc17.cc

/// @page txc18.cc txc18.cc
/// @include txc18.cc

/// @page tictoc13.msg tictoc13.msg
/// @include tictoc13.msg

/// @page tictoc14.msg tictoc14.msg
/// @include tictoc14.msg

/// @page tictoc15.msg tictoc15.msg
/// @include tictoc15.msg

/// @page tictoc16.msg tictoc16.msg
/// @include tictoc16.msg

/// @page tictoc17.msg tictoc17.msg
/// @include tictoc17.msg

/// @page omnetpp.ini omnetpp.ini
/// @include omnetpp.ini

};
