### Debugging Actions



### Standard debugging commands of the StDebugger
The debugging commands offered by the StDebugger in the commands toolbar are the following ones:

* Proceed: Resumes the normal execution of the debugged process until a new breakpoint is hit or the program is finished.

* Into: Steps into method invocations.
Interprets the bytecode instructions of the program until a message is about to be sent,  a method is activated (\ie{}after the message send), a method is about to return, or an assignment is about to be performed.

* Over: Steps through the code, avoiding entering method invocations or closure evaluations.
Interprets the bytecode instructions of the program.
Stops only for the following conditions in the current \texttt{Context}\footnote{In Pharo, a \texttt{Context} is an object representing what is commonly known as a stack frame, containing data of the execution state of a method and a pointer to the calling frame (the sender \texttt{Context})} or its sender context: until a message is about to be sent, a method is about to return, or an assignment is about to be performed.

* Through: Steps through the code, avoiding entering method invocations. 
It enters \texttt{BlockClosure}s\footnote{In Pharo, a \texttt{BlockClosure} is an object that represents closures, used as anonymous functions or code blocks.} defined in the current activated method.
Interprets the bytecode instructions of the program.
Stops when a message is about to be sent, a method is about to return, or an assignment is about to be performed in specific contexts.
These contexts are: the current context, any \texttt{BlockClosure} context originating from the current context, or its sender context.

* RunTo: Executes instructions up to the code under the caret or until the current method returns.

* Restart: Goes back to the start of the context currently selected in the stack, typically the current executing one, reinitializing its local variables.

### How and where are debugger actions implemented