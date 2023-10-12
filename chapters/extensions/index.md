## Extending the debugger
The debugger is extendable with user-defined problem-specific debugging tools.
Both views and debugging model can be tweaked to become applicable to a user's problem.
Views are extendable through the moldability of the Spec framework.
The Pharo debugging model is accessible through the Sindarin debugging API, with which users write debugging scripts and extend debugging actions.

Different aspects of the debugger are extendable: the toolbar, contexts menus (stack, code editor and context inspector) and the right column.

!!!The debugger tools extensions
The right column extends the debugger with additional tools.
These tools have access to the debugging model features and state through the Sindarin API (Chapter **Sindarin**).
They use this API to provide customized, problem-specific actions and views of the debugged execution.
When a debugging extension is loaded in the system and activated, it automatically appears in the debugger when it opens (if default layouts are used).

++SCREENSHOT HERE++




!!!!Implementing a debugger extension
First, a spec presenter must be implemented.
It will be the view displayed in one tab of the extension column.
Spec presenters are subclasses of ==SpPresenter==.
These presenters must implement specific methods and state to be considered as valid extensions.
These methods are described below:


""==debuggingExtensionToolName==.""
This is a class-side method containing a specific pragma that defines the presenter as an extension.
This method must return a string that will be the title of the extension tab in the debugger.
An example is shown in the following script for the bytecode extension of the debugger.

[[[
StSindarinBytecodeDebuggerPresenter class >> debuggingExtensionToolName
	<debuggerExtensionOrder: 2 showByDefault: true>
	^ 'Bytecode'
]]]

The pragma actually enables instances of that presenter to be used as a debugger extension.
It has two parameters:
- the order in which the view will appear in the debugger extension column tabs
- a boolean that shows or not the extension by default in the debugger

Once the pragma defined in this method, these two parameters automatically appear in the settings under ==Tools >> Debugging >> Debugger Extensions==.
An additional setting named ''activate debugger extensions'' switches on and off all debugger extensions.


""==setModelBeforeInitialization: aStDebugger==.""
This is an instance-side method.
It is automatically called by the debugger when instantiating the extension view before initializing it.
It passes a model as parameter, that is always the current debugger for which the extension view is instantiated.
That method must, if needed, use the debugger to retrieve whatever model the extension needs.
For example (see code below), the bytecode extension needs the debugger itself and an instance of a Sindarin debugger to script the execution.
Developers must implement the needed instance variables and methods for each extension accordingly.

[[[
StSindarinBytecodeDebuggerPresenter >> setModelBeforeInitialization: aStDebugger
	"My original model is the debugger presenter that I extend"
	stDebugger := aStDebugger.
	sindarinDebugger := aStDebugger sindarinDebugger
]]]


""==updatePresenter==.""
This is an instance-side method.
When the debugger intercepts an event of interest, the debugger enumerates activated extensions and calls the ==updatePresenter== method on each of them.
The example below shows the bytecode extension ==updatePresenter== method implementation.
As it is a method inherited from the spec framework hierarchy, it is recommended to always send that message to super.

[[[
StSindarinBytecodeDebuggerPresenter >> updatePresenter
	super updatePresenter.
	self updateBytecode
]]]

!!!!Interaction between debugging extensions and the debugger

Tools extensions are instantiated and registered by the debugger.
When the debugger is updated, ''e.g.'', while a single step has been performed, all registered tools are notified of that update.

Tools extensions might need to notify the debugger that an update of all views is needed.
To trigger a general refresh of all views, tools must send the ==#forceSessionUpdate== message to the debugger.

Tools might also have to turn the debugger updating service off while they act on the current debug session.
This avoids systematic refresh of the debugger view while they work which can considerably slow down the execution (''e.g.'', while doing multiple steps).
To turn off the debugger refresh service, tools must send the ==#removeSessionHolderSubscriptions== message to the debugger.
To turn it on, tools must send the ==#setSessionHolderSubscriptions== message to the debugger.

In those cases, developers must make sure that their extensions hold a reference to the debugger.
This is done by declaring an instance variable in the extension class, and set that variable in the ""==setModelBeforeInitialization: aStDebugger=="" method body.

When a specific tool updates the debugger, the debugger also notifies all registered extensions.


!!!Context menus and toolbar extensions

Context menus and toolbar extensions are either standalone (they perform a user-defined action or debugging script) or associated to a tool extending the debugger.

!!!!Standalone: Sindarin script or custom user-defined action
Give the example of debugging scripts.

!!!!Tool-related extension

Extensions are collected through pragmas
The debugger toolbar is extendable by developing a command class with the <debuggerToolbarExtension>
The debugger inspector context menu is extendable by developing a command class with the <debuggerToolbarExtension>
