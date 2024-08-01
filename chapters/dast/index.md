## The Debuggable AST Interpreter

### How to use the interpreter

This section was written, based on this version of Pharo: `Pharo-13.0.0+SNAPSHOT.build.144.sha.ac276f94e7831a3329f686b3a6fa842ca3c8832e (64 Bit)`

#### Load the interpreter

To load the AST interpreter only, open a playground and execute this code to load the baseline:

```Smalltalk
Metacello new
    baseline: 'DebuggableASTInterpreter';
    repository: 'github://adri09070/DebuggableASTInterpreter:Oups-DAST-handling';
    load.
```

#### Example

In a playground, instanciate an interpreter and initialize it with the program you want it to interpret (here: `Point x: 1 y: 2`):

```Smalltalk
interpreter := DASTInterpreter new.
interpreter initializeWithProgram: (RBParser parseExpression: 'Point x: 1 y: 2').
```

You can see the stack of AST nodes the current context still has to interpret by inspecting: `interpreter currentContext nodes`. The nodes will be interpreted from last to first.

_Node stack:_  
![Initial Node stack](./figures/interpreter-example-initial-nodes.png width=70)


You can see the value stack of the current context (where the interpreter pushes the values of the AST nodes it interprets) by inspecting: `interpreter currentContext stack`. It is empty at the moment because nothing has been evaluated yet.


Evaluate `interpreter stepInto` 3 times to evaluate the receiver and arguments of the `#x:y:` message being interpreted.  

_New node stack:_  
![Node stack after stepping into](./figures/interpreter-example-nodes-after-step-into.png width=70)  

_New value stack:_  
![Value stack after stepping into](./figures/interpreter-example-value-stack-after-step-into.png width=70)  


Finally, the message send itself is ready to be interpreted. If you evaluate `interpreter stepOver`, the interpreter will pop the receiver and arguments from the value stack, evaluate the message send completely, and push its value on the value stack.  

_New node stack:_  
![Node stack after stepping over](./figures/interpreter-example-nodes-after-step-over.png width=70)  

_New value stack:_
![Value stack after stepping over](./figures/interpreter-example-value-stack-after-step-over.png width=70 )  

As you can see, the value of the message send is the point it created: `(1@2)`

### How to use the DAST debugger

What is described in this section does not work for this version of Pharo: `Pharo-13.0.0+SNAPSHOT.build.144.sha.ac276f94e7831a3329f686b3a6fa842ca3c8832e (64 Bit)`. 
Maybe this could work by merging some waiting PRs on the _RMODInria DebuggableASTInterpreter_ Github repository.

It works in `Pharo-11.0.0+build.726.sha.aece1b5473acf3830a0e082c1bc3a15d4ff3522b (64 Bit)`

#### Load the debugger

To load the AST interpreter with its debugger, open a playground and execute this code to load the baseline:

```Smalltalk
Metacello new
    baseline: 'DebuggableASTInterpreter';
    repository: 'github://adri09070/DebuggableASTInterpreter:Oups-DAST-handling-p11';
    load: #DebuggerXP.
```

#### Example

Open a class browser on the class `SindarinDebuggerTest`.
Right-click on its method `methodWithTwoAssignments`, and set a DAST breakpoint on the method via the context menu `Breakpoints > DAST breakpoint`

![Setting a DAST breakpoint on a method](./figures/debugger-example-set-dast-breakpoint.png width=70)

Then, open a playground, try to execute this code:

```Smalltalk
SindarinDebuggerTest new methodWithTwoAssignments 
```

This will open the DASTDebugger on the method `SindarinDebuggerTest>>#methodWithTwoAssignments`:

![DASTDebugger opening](./figures/debugger-example-dast-debugger-open.png width=70)

You can observe what the interpreter does if you click on _Step over_ 7 times. 
This will execute all code related to the next 7 AST nodes that has bytecodes associated to it.
The debugger now stops on the message send `#x:y:`:

![DASTDebugger after 7 step overs](./figures/debugger-example-after-step-over.png width=70)

Now, you can click on _Step_ to enter the method `Point>>#x:y:`:

![DASTDebugger after step](./figures/debugger-example-after-step.png width=70)

Then, it is possible to select the context `SindarinDebuggerTest>>#methodWithTwoAssignments` in the context stack and click on the _Restart_ button.
This will restart completely the execution of this context, starting from the creation of the method's temporary variables:

![DASTDebugger after restart](./figures/debugger-example-after-restart.png width=70)

Finally, you can click on _Proceed_, which will execute the program to the end using the DAST interpreter.

#### Second example: switch from StDebugger to DASTDebugger and vice-versa

In a playground, paste this code and debug it:

```Smalltalk
| debuggerObjectForTest block a b |
				debuggerObjectForTest := StDebuggerObjectForTests new. 
				a := 0.
				block := [ | c | debuggerObjectForTest methodWithTempsAssignments. 1=2 ifTrue: [ block value ]. a := a + 1. b:=0. c:=0 . [ a:= a + 2. c:=c+4. b:=b+3 ] value ].
				^ block value + b + a
```

Click on _Over_ once in the `StDebugger` then click on the button with an exclamation mark icon in the top-right corner:

![Switch from StDebugger to DAST](./figures/switch-example-dast-debugger-opening.png width=90)

This will close the StDebugger and open the DAST debugger on the same program, at the same instruction, which will be executed with the DAST interpreter.
Click on _Step Through_ 8 times to enter the block.
Click on _Step Over_ 17 times and on _Step Through_ again to enter the block `[ a:= a + 2. c:=c+4. b:=b+3 ]`.
Click on _Step_Over_ 10 times to stop on this instruction `b + 3`.
Now, you can click on the grey button above the context stack in the DAST debugger:

![Switch from DASTDebugger to StDebugger](./figures/switch-example-stdebugger-opening.png width=70)

This will reopen the `StDebugger` to the same instruction we have stopped to in the `DASTDebugger`.
You may now proceed the execution to terminate the program.

#### Known problems:

- When proceeding a program in the DASTDebugger, the program will terminate, but an exception will be raised, opening an `StDebugger`. I think this could easily be fixed.

- Switching from the Pharo interpreter to the DAST interpreter raises an exception when the StDebugger highlights an entire `DoIt` method. I think this could easily be fixed.

- Switching from the Pharo interpreter to the DAST interpreter (and vice-versa) raises an exception when either interpreter is stopped on code that is optimized in Pharo (`ifXXX:` messages, `whileXXX:` messages, `to:do:`, etc.) We could not find any solution to this problem...

- The code highlighting may be completely messed up in the DAST debugger...

- "Switch to StDebugger" button is ugly in DASTDebugger

