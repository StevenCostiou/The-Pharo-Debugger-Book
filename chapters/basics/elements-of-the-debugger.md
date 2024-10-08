### Elements of the Debugger

In this chapter, we describe the elements of the debugger graphical user interface, shown in the following figure. 
From top to bottom, the debugger shows an option toolbar, a call stack, a command toolbar, a code pane and an inspector.

![The debugger GUI.](graphics/debugger-elements.png)

#### The option toolbar

![The debugger options toolbar.](graphics/debugger-elements-options.png)

**Extensions.**
The _extensions_ option opens a popup to choose which debugger extension to activate or deactivate.
Debugger extensions are covered later in the corresponding chapter.

**Filter debugging information.**
Filters debugging information in the stack.
The filter hides unnecessary low-level debugging concerns that otherwise appear in the context stack, such as additional contexts in the debugging core.
Clicking on this button toggles the filter on and off.

**Configuration.**
Opens the settings browser scoped on the debugging options.
From this browser, we can configure the debugger, its infrastructure and tools.
The debugger configuration is covered in the corresponding chapter.

#### The stack
The stack is a visual representation of what, in general terminology, is known as an execution call stack.
Each element in the stack is an object called a *context*, that represents a method being executed.
Each context has a receiver, *i.e.*, the object executing the method, and a sender, *i.e.*, another context from which a message was sent to the receiver, provoking the execution of the method by the receiver.

The last context in the stack (at the top) is the current executing context, where the execution is interrupted and waiting for control. The first context in the stack (at the bottom) is the context from which the execution began, which triggered the successive context call chain until the execution interruption.

![The stack, with its context call chain.](graphics/stack.png)

The stack has a context menu accessible by right clicking on any element of the stack. This menu provides different interactions with the selected stack element: 
- **Return:** Return of a given value to the previous context.
- **Copy Stack to Clipboard:** Copy the debugging stack to the clipboard.
- **Peel to first like this:** Peels the stack back to the second occurance of the currently selected message.
- **File out method:** Write the source code of the selected context's method on a file.
- **Browse all instances:** Browse all instances of the class of the selected context's receiver.
- **Browse all subinstances:** Browse all instances of the selected context's receiver class and all its subclasses.
- **Browse pointers to:** Browse all objects referencing the receiver of the selected context.
- **Browse method inheritance:** Browse the hierarchy implementors of the selected method
- **Browse method versions:** Browse the version history of the method.
- **Implementors of it:** Browse all implementors of the selected method or message
- **Senders of it:** Browse all senders of the selected method or message
- **References to it:** Browse all references to the selected method or selector
- **Browse all instances:** Browse all instances of the selected class
- **Browse all subinstances:** Browse all instances of the selected class and all its subclasses
- **Browse class references:** Browse all references to the receiver's class
- **Browse class hierarchy:** Browse the class hierarchy of the selected class
- **Browse class variables:** Browse the class variables
- **Browse class variables reference:** Browse the references to class variables
- **Browse instance variables reference:** Browse the references to instance variables
- **Browse pointers to:** Browse all objects referencing the selected object


#### The commands.

Developers manually step through the execution by using the commands (\autoref{fig:stdebugger}~b).
After any debugging action, the code presenter (\autoref{fig:stdebugger}~c) is updated and highlights the source code that will be executed in the next step.
Developers observe execution data using the inspector (\autoref{fig:stdebugger}~d) in addition to the stack.
%It allows developers to control a program thread execution (a `Process}) which is advanced by performing \textit{stepping operations} that are available in the debugging commands toolbar.

Commands are actions applicable to the current context
After each debugging action (''e.g.'', a step), the list of commands is being updated.
New commands may appear or disappear.
It depends if they are applicable or not to the current context and debugging session.
For instance, when a ==doesNotUnderstand== exception is raised (''i.e.'', a method does not exist), the command ''createMethod'' will appear.
In that case, it gives users the possibility to create the missing method and continue the execution.

The debugging commands offered by the StDebugger in the commands toolbar are the following ones:

* Proceed: Resumes the normal execution of the debugged process until a new breakpoint is hit or the program is finished.

* Into: Steps into method invocations.
Interprets the bytecode instructions of the program until a message is about to be sent,  a method is activated (\ie{}after the message send), a method is about to return, or an assignment is about to be performed.

* Over: Steps through the code, avoiding entering method invocations or closure evaluations.
Interprets the bytecode instructions of the program.
Stops only for the following conditions in the current `Context`. 
In Pharo, a `Context` is an object representing what is commonly known as a stack frame, containing data of the execution state of a method and a pointer to the calling frame (the sender `Context`) or its sender context: until a message is about to be sent, a method is about to return, or an assignment is about to be performed.

* Through: Steps through the code, avoiding entering method invocations. 
It enters `BlockClosures` defined in the current activated method.
In Pharo, a `BlockClosure` is an object that represents closures, used as anonymous functions or code blocks.
Interprets the bytecode instructions of the program.
Stops when a message is about to be sent, a method is about to return, or an assignment is about to be performed in specific contexts.
These contexts are: the current context, any `BlockClosures` context originating from the current context, or its sender context.

* RunTo: Executes instructions up to the code under the caret or until the current method returns.

* Restart: Goes back to the start of the context currently selected in the stack, typically the current executing one, reinitializing its local variables.

#### The code.

""The code.""
The code pane shows the source code of the interrupted context's method.
The source code is annotated with informations about the position of the interrupted execution.
This position corresponds to the next execution that will be executed if a step action is performed.
It is coloured in blue if that instruction is selected in the source code, or in orange and underlined if the selection is empty.

A context menu (through right click) provides inspection, exploration, and stepping actions applicable to the context.
Such actions include, ''e.g.'', browse implementors of a selected message or skip execution of selected parts of the code.


#### The inspector.



""The inspector.""
It provides views on the current context.
It contains tabs with different inspections or views on the current context or on the system.

The raw inspection shows three items: ''Receiver'', ''Temps'' and ''Context''.
The receiver item shows information about the receiver of the message, or the object executing the method from the context.
The temps item shows the temporary variables of the current context.
Finally, the context item shows lower-level details, such as the interrupted context, the debug session or the debugger itself.
Double-clicking on an item will spawn a new view in the same inspector, with an inspection on the selected item.

The breakpoints inspection shows all breakpoints installed for the current receiver.
Those breakpoints are either installed on the receiver's class (''e.g.'', halts) or on that specific receiver (''i.e.'', object-centric breakpoints or halts.)
Ticking or unticking a breakpoint in that list will activate or deactivate that breakpoint.

The meta inspection shows methods of the receiver's class hierarchy.
Methods can be modified and saved in directly in that pane.

