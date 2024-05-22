## Architectural Aspects

### The Debugger Infrastructure
![Class Diagram of the Pharo Debugger Infrastructure.](./graphics/debugger-class-diagram.jpeg width=75&label=fig:debugger-class-diagram)

In this section, we briefly describe the architecture of the Pharo debugger infrastructure, shown in *@fig:debugger-class-diagram@*.

#### A Modular Infrastructure

The debugger components can be classified into 3 categories.
They are either a part of the debugger logic, a part of its model or a part of its GUI.
The logic of the debugger is implemented in the _pharo-project_ Github repository whereas the model and the GUI of the debugger are implemented in _Newtools_

Through misuse of language, we generally consider that the debugger is the debugger GUI, called the `StDebugger`.
This user interface uses an object called an `StDebuggerActionModel` that implements the API of the debugger. 

This debugger action model uses some `StDebuggerContextPredicate`s, which are some objects that determine the type of execution that is debugged.
These context predicates allow the debugger GUI to display only the actions that are available at the moment. For instance, the debugger should allow to create a missing method only if the debugger opened on an exception due to a missing method in a class.

The debugger action model also uses a `DebugSession` that implements the debugger logic.
This debug session uses itself an interpreter (an `InstructionStream`) to execute a program, step by step.
When a step has been performed, the debug session notifiies its debugger action model which informs its GUI that it should update itself.

This infrastructure is modular because it allow to interchange a component with another. 
For example, we can change the interpreter with another interpreter, and the same goes for the action model or the GUI

#### An Extensible Infrastructure

The debugger infrastructure is also extensible.

Indeed, it is capable of dynamically changing its _layout_.
So, this is possible to customize the user experience by changing the appearance of the debugger, without closing an old debugger to open a new debugger, which would be a source of frustration for the user.

The Pharo debugger is also extensible through meta-programming.
For instance, just by using a trait, the debugger knows which tool can be activated as a debugger extension.
A debugger extension is a tool that is directly integrated to the debugger that displays additional information/actions to help debug your program.
From the moment a debugger extension has been created, it is automatically added to the system settings and it is already possible to activate it in order to display it in the debugger.

Another example assessing this extensibility via metaprogramming is the ease with which it is possible to add debugging commands.
Indeed, in addition to the basic debugging commands present in the debugger toolbar, other advanced debugging commands use a specific pragma, which is an annotation on a method to attach additional properties to it. This eases the collection of these methods via _reflectivity_.
These commands are written with an API called _Sindarin_.
_Sindarin_ provides both low-level and high-level APIs to easily write debugging scripts, such as _step until the current method returns_ or _step until a specific message is sent to a specific object_.


## Design Choices

### The Debugger Extension Design

![Class Diagram of the New Debugger Extension System.](./graphics/debugger-extensions-class-diagram.jpeg width=75&label=fig:debugger-extension-class-diagram)

Everything that follows is a description of the class diagram in *@fig:debugger-extension-class-diagram@*.

#### An Old Static Debugger Extension System 

Previously, the debugger extension system was static.
A debugger extension uses the trait `TStDebuggerExtension`. It mostly defines two parameters class-side:

- its _display order_, which determines if it should display before or after another extension,
- its `showInDebugger` setting, which determine if the debugger extension is activated or not.

When the `showInDebugger` setting of a debugger extension changes, the `SystemAnnouncer` raises a `StDebuggerExtensionActivationToggle` announcement.
When a debugger opens, it collects all activated debugger extensions and instanciates them to add them to its collection of extension tools and to add a page in the debugger extension notebook for each. 
As each debugger subscribes to the `SystemAnnouncer` to receive `StDebuggerExtensionActivationToggle` announcements, they can update their collection of extension tools and their debugger extension notebook by removing/adding the concerned extension if it is now (de)activated.
Then, **all** debuggers that would open after the announcement has been made would show/hide this extension.

As a result, it is not possible to display debugger extensions in **some** debuggers but not all of them.
Indeed, if developers developed a debugger extension to help them debug their domain-specific application, they want their extension to appear only when they are debugging their application.
If their extension is shown when they're not debugging their application, the behavior of their extension may be undefined and it could result in a debugger crash or a crash of the Pharo image.

Moreover, we felt it could be practical to perform some debugging actions that aren't domain-specific in context-specific debugger extensions.

#### A Debugger Extension System Allowing Context-Specific Debugger Extensions

To allow context-specific extensions, the debugger extension determines if it can be displayed in a debugger via the method: 

```pharo
TStDebuggerExtension>>#acceptsPredicate:  aStDebuggerContextPredicate

   ^ true "By default, a debugger extension is static"
```

This method uses the debugger's context predicate to determine if the extension should be displayed or not. **We could discuss: Should the method use a debugger as parameter instead of a context predicate?** The extension would have more parameters to decide if it should display itself or not, but is it necessary?

This induces two big changes in the update of extensions:
- When an extension is (de)activated which sends an `StDebuggerExtensionToggleAnnouncement` to the debugger, the debugger now should consider the current context predicate,  because if it is refused by the toggled extension, the extension toggle should be ignored: it has no effect
- The context predicate can change after each action in the debugger, including steps (correct me if I'm wrong). So the extensions should be updated accordingly. Instead of just updating the instanciated presenters, we should check if there are:
  * extensions that were activated but that didn't accept the debugger context predicate until now, in which case it should be added now.
  * extensions that were activated and that did accept the debugger context predicate until now, in which case it should be removed from the extension tools.

This can be summed up by the two following boolean tables.

**Boolean table describing the action to take according to the current debugger context predicate when a debugger extension has been (de)activated, resulting in a `StDebuggerExtensionToggle` announcement:**

| `showInDebugger` state change | accepts predicate | action on extension |
| ---------------------------------| ------------------ | -------------------- |
| / | `false` | nothing |
| `false` -> `true` | `true` | **display** |
|  `true` -> `false` | `true` | **hide** | 

**Boolean table describing the action to take according to the current `showInDebugger` state of an extension, when  an action has been performed in the debugger, resulting in a potential change of status in the debugger context predicate:**

| `showInDebugger` state | accepts predicate change | action on extension |
| -------------------------| -------------------------- | -------------------- |
| `false` | / | nothing |
| `true` | `false` -> `false` | nothing |
| `true` | `true` -> `true` | nothing |
| `true` | `false` -> `true` | **display** |
| `true` | `true` -> `false` | **hide** |

#### Illustrating the New Debugger Extension System With an Example

![Sequence Diagram of the New Debugger Extension System.](./graphics/debugger-extensions-sequence-diagram.jpeg width=75&label=fig:debugger-extension-sequence-diagram)

For better understanding, I now describe an example illustrated by the sequence diagram in *@fig:debugger-extension-sequence-diagram@*.
In this example, the user created a debugger extension named `aDebuggerExtension`.
Its `showInDebugger` setting is initially set to `false`.

A debugger opens which initializes itself so it subscribes to its debugger action model that creates the initial debugger context predicate. 
Then, it initializes all the extension tools that are activated and that accepts this debugger context predicate.
The debugger also subscribes to extension (de)activations.

Then, the user activates its debugger extension (i.e: `showInDebugger` setting is set to `true`), which causes the debugger to update itself with the messsage `#updateExtensionsFromAnnouncement:`. The debugger sends the message `#acceptsPredicate:` to the extension class, which returns `false` and does nothing as a result (line 1 in the first boolean table).

After that, the user performs a step in the debugger, which causes the debugger action model to update its context predicate and to update itself with the message `#updateStep`. 
When updating its debugger extensions, the debugger collects the extensions that should be displayed (i.e: all activated extension classes that accepts the current context predicate).
To do that, it sends the message `#showInDebugger` that returns `true` for the user's debugger extension class. As it returns `true` for this extension class, the debugger sends the message `#acceptsPredicate` for the same debugger extension class, which also returns `true`.
Therefore, `aDebuggerExtension class` is added to the extensions to display.
The debugger checks the currently registered tools. 
As this extension is currently not displayed, it is added to the debugger's extension tools and displayed in this debugger (line 4 in second boolean table).

Later, the user deactivates its debugger extension.
So, the debugger receives again the message `#updateExtensionsFromAnnouncement:` that sends again the message `acceptsPredicate:` to this debugger extension class. 
As it returns `true`, the debugger sends the message `#showInDebugger` to this debugger extension class.
This returns `false` so the debugger removes the extension from its extension tools and hides it (line 5 in second boolean table).

This example does not cover every case described in the previous boolean tables but should be enough to understand as the cases that aren't covered are symmetrical to the cases that are covered.

### The Debug Point Model Design

The goal of _Debug Points_ is to have configurable breakpoints that can have different _behaviors_.
They are called debug points because when a user instrument some code to debug, it does not necessarily want to open a debugger on the target code location. Following this logic, breakpoints and watchpoints are regrouped into the family of _Debug Points_.

#### Initial Design

![Class Diagram of the _Debug Points_ Model Before Rework](./graphics/dp-class-diagram-old.jpeg width=75&label=fig:dp-class-diagram-old)

Initially, _Debug Points_ are the [work of an intern called Max Zurbriggen](https://github.com/Max-Zurbriggen/PharoDebugPoints), a student of the University of Zurich.
We describe the model he conceived and implemented, shown in *@fig:dp-class-diagram-old@*

A debug point has a _name_ so that the user can identify them, and targets an _AST node_.
When installed, it installs a _metalink_ on this node that executes the method `hitWithContext:` of this debug point each time just before the node is reached.
A debug point also has a target instance that is `nil` by default. If it is not `nil`, the debug point becomes _object-centric_.

A debug point contains _behaviors_ that can be added or remove to configure a debug point.
A _behavior_ knows its debug point, has a numbered priority and defines a method `execute` to execute a behavior that should be executed when its debug point is hit. This method returns a boolean that determines whether the corresponding debug point should be executed or not.
Each _behavior_ subclass can hold state useful for their execution.
There are several types of debug point behaviors: a behavior that disables its debug point after it has been hit _once_, a behavior that _counts_ the number of times its debug point has been hit, a behavior that checks if a _condition_ of the execution is true, a behavior that executes a _script_ and much more.

These _behaviors_ are all executed each time a debug point is hit, ordered by decreasing priority.
However, if a _behavior_ returns _false_, the behaviors that should have been executed afterwards and its debug point are not executed.
If all _behaviors_ return `true`, debug points execute their specific code: breakpoints open a debugger while _watchpoints_ save the value of the attached _AST node_.

Finally, all debug points define their `uiElement` class that should be used to display information and/or configure the debug point/_behavior_ in a GUI called the _Debug Point Browser_.

#### Design at Release, After Rework

![Class Diagram of the _Debug Points_ Model After Rework](./graphics/dp-class-diagram-new.jpeg width=75&label=fig:dp-class-diagram-new)

We describe the improvements we provided to the previous model to fix its weaknesses.
The two following paragraphs illustrate the class diagram shown in *@fig:dp-class-diagram-new@*.

##### Debug Point Targets

A problem with the _Debug Point_ model described previously is that the target of a debug point can only be a _node_.
They cannot target _variables_ like in the former breakpoint model and implementing it while keeping this model would result in installing dozens of debug point for each AST node that access the target variable, which is overwhelming.

So, the goal is to be able to perform the same actions on different entities (i.e: AST nodes and instance variables, at least).
To do this in object-oriented programming, nothing is better than **abstractions**.
Therefore, I created an abstract class `DebugPointTarget` that define what should be specified by subclasses: basically what should be done differently between the subclasses.
In our case, what is different between a node and a variable is: 
	
- the API used to install a metalink on it,
- the API to access the _target class_ (e.g: the class of the method represented by the method node of an AST node VS the class defining an instance variable),
- the API used to _browse_ the class of the target

In the end, a debug point _target_ is a wrapper for the real target (i.e: a node or a variable) that allows to define these abstractions.
The subclasses then implement these abstractions.
One subclass wraps an AST node whereas another subclass wraps an instance variable.
This solution has the advantage to be extensible as I defined only once what is common between nodes and variables and to create a new type of debug point target, one would just need to create a subclass of debug point _target_ that implements these abstractions.
As a result, to set a debug point on a variable, the user only see one to instead of dozens of smaller ones, which can easily be (de)activated.

These abstractions are defined directly in the `DebugPointTarget` class but also in an intermediary abstract class `DebugPointClassTarget`.
We implemented it this way to be able to _decorate_ it, following the _Decorator_ design pattern.
This works well because both AST node and variables can be decorated to be object-centric with a `DebugPointObjectTarget`.
The only difference between an object target is the way to install the metalink. The object-centric decorator installs its subtarget on the wrapped object, while a class target install it on its target class.
This also adds an abstraction to define the _scope_ of a target (i.e: class or target instance).
This is better than the model previously described as the target instance could be `nil`, which would always imply to do a check that this is not the case.
Moreover, for object-centric ddebug points, the former model used to check that the receiver was identical to the target instance each time the debug point was hit, which is slow and not object-oriented.
Our implementation take advantage of the API of metalinks that is the same to install one on a class or on an object, and that is optimized.

##### Debug Point Behavior Refactoring

Another problem with the old _Debug Point_ model is the way debug point _behaviors_ are implemented.
Indeed, by observing these behaviors, it appears there are clearly two types of behaviors:

- behaviors that perform conditional checks and that return the result,
- behaviors that execute code that perform side-effects on the current execution and that always return `true`.

This design is weird as a good principle in object-oriented programming is to only have methods that return something or methods that have side-effects but not methods that do both.

Moreover, this can cause some problems because, in the old model, if a behavior that performs side-effects has a higher priority than a behavior that performs a check that would return `false`, then the side-effects would have been done even though they should not have.
Indeed, if a condition return `false`, then it means that the debug point should not be hit, and notably that no side-effect should be applied.
For instance, if a condition behavior of a debug point returns `false`, the count behavior of the same debug point should not increase its counter.

To make a clear separation between both, we added two intermediary abstract classes of debug point _behaviors_ for both types.
Debug points now hold these behaviors in two different collections and the corresponding behavior abstract classes define in (resp. from) which list their instances should be added (resp. removed).

## Testing the Debugger