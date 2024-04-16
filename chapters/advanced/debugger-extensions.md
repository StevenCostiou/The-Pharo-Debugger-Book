### Debugger Extensions

Debugger extensions are tools that are directly integrated to the debugger that display additional information/action to help debug your program.

When activated, these extensions appear in the right pane of the debugger, as a page of a notebook (Figure *@fig:debugger-extension-example@*).

![Example of debugger extension displayed in the right pane](./graphics/debugger-extension-example.png width=90&label=fig:debugger-extension-example)

#### Debugger Extension Settings

Debugger entension settings can be found in the Pharo system settings, which can be opened by pressing _CMD+O+S_ on MAC OS or _CTRL+O+S_ on Windows/Linux.
Each debugger extension has a dedicated submenu in these settings at the following path: `Tools > Debugging > Debugger Extensions`. For each extension, can be configured two settings: 
- the position at which it will be displayed, set to first as default and referred as 1 in Figure *@fig:debugger-extension-system-settings@*. All extensions that are at position 1 will be displayed before all extensions at position 2, and so on.
- the setting to show or hide the extension in the debugger. A debugger extension is hidden as default. This setting is referred as 2 in Figure *@fig:debugger-extension-system-settings@*. When a debugger is shown/hidden, it is dynamically shown/hidden in all debuggers that are already open if they can display the (un)activated debugger extension. 

![The debugger extension settings in the system settings](./graphics/debugger-extension-system-settings.png width=90&label=fig:debugger-extension-system-settings)

As it may be tedious to go deep into the system settings to show or hide a debugger extension, a shortcut allows to show/hide a debugger extension from a debugger directly. These settings can be open via a popup by clicking the first button of the action bar, located above the context stack (Figure *@fig:debugger-extension-settings@*).

![The debugger extension settings in the debugger](./graphics/debugger-extension-settings.png width=90&label=fig:debugger-extension-settings)

#### How to create a debugger extension?

To create a debugger extension, you need to create a class that inherits from `SpPresenter` and that uses the trait `TStDebuggerExtension` (Figure *@fig:debugger-extension-creation@*).

![Creation of the debugger extension class](./graphics/debugger-extension-creation.png width=70&label=fig:debugger-extension-creation)

First thing to do, a debugger extension needs to redefine the **method `#extensionToolName`**. This method should return a string that will refer to your debugger extension, in the debugger and in the system settings (Figure *@fig:debugger-extension-name@*).

```language=Pharo&caption=Defining the name of your debugger extension&label=fig:debugger-extension-name
StMyDebuggerExtension>>#debuggerExtensionToolName
	
	^ 'My debugger extension'
```

By default, a debugger extension is not shown in the debugger. This behavior is defined by the **class-side method `#showInDebugger`** (Figure *@fig:debugger-extension-show@*). To activate the debugger extension by default, you should override this method in your debugger extension class.

![A debugger extension is hidden by default](./graphics/debugger-extension-show.png width=70&label=fig:debugger-extension-show)

When activated, A debugger extension is displayed first by default. To change that, you should redefine the **class-side method `#defaultDisplayOrder`** (Figure *@fig:debugger-extension-display-order@*).

![A debugger extension is displayed first by default](./graphics/debugger-extension-display-order.png width=70&label=fig:debugger-extension-display-order)

If a debugger extension class is activated, it won't necessarily show itself in every debugger. 
Indeed, in order to be displayed in a debugger, the debugger extension needs to determine whether it accepts this debugger's context predicate.
A debugger context predicate is an object provided by the debugger and determines the type of execution context the debugger is executing. 
The API of the `StDebuggerContextPredicate` class (Figure *@fig:debugger-context-predicate@*) can be used in the **class-side method `#acceptsPredicate:`** of your debugger extension.

![Debugger context predicate's API](./graphics/debugger-context-predicate.png width=90&label=fig:debugger-context-predicate)

By default, a debugger extension can be shown by all debuggers.
In our example, we will override this method so that our debugger extension is displayed only when an exception has been signaled in the debugged execution (Figure *@fig:debugger-extension-predicate@*).


```language=Pharo&caption=Example of usage of method #acceptsPredicate&label=fig:debugger-extension-predicate
StMyDebuggerExtension>>#acceptsPredicate: aStDebuggerContextPredicate

	^ aStDebuggerContextPredicate contextSignalsException
```

In this example, we will create a debugger extension that opens an inspector on the exception if the debugged context signaled an exception.
Just like any presenter, you need to initialize your subpresenters (Figure *@fig:debugger-extension-initialize@*) and define a default layout (Figure *@fig:debugger-extension-layout@*).

```language=Pharo&caption=Initializing subpresenters of your debugger extensionl&label=fig:debugger-extension-initialize
StMyDebuggerExtension>>#initializePresenters

	inspector := StRawInspectionPresenter new
```

```language=Pharo&caption=Defining a default layout for your debugger layout&label=fig:debugger-extension-layout
StMyDebuggerExtension>>#defaultLayout

	^ SpBoxLayout newTopToBottom
		  add: inspector;
		  yourself
```

A debugger extension can define a layout specifically to be displayed in the debugger. 
By default, the layout that is used in the debugger is the default layout (*@fig:debugger-extension-debugger-layout@*). 
However, if you want to use a different layout in the debugger than the default one, you should override the method `#debuggerLayout`.

![A debugger extension's debugger layout](./graphics/debugger-extension-debugger-layout.png width=70&label=fig:debugger-extension-debugger-layout)

Now that you defined your subpresenters and the layout of your debugger extension, you should define how it should update after an action has been taken in the debugger.
This must be defined inside the **method `#updatePresenter`**.
In our case, we just set the inspector model to the exception of the debugger (Figure *@fig:debugger-extension-update@*). This can be recovered because, by using the trait `TStDebuggerExtension`, **any debugger extension can access its attached debugger via the instance variable `debugger` or via a getter of the same name**.

```language=Pharo&caption=Defining how a debugger extension should update after each action in the debugger&label=fig:debugger-extension-update
StMyDebuggerExtension>>#updatePresenter

	inspector model: debugger exception
```

The debugger extension is now finished. By creating only 5 small methods, if you activate your debugger extension in the settings, your exception will display in the debugger with an inspector on the exception, only if an exception has been signaled (Figure *@fig:debugger-extension-finished@*).

![The new debugger extension is now displayed in the debugger when an exception is raised](./graphics/debugger-extension-finished.png width=90&label=fig:debugger-extension-finished)

##### More advanced features

A debugger extension can be integrated more deeply in the debugger than just in the right pane.

Indeed, a debugger uses `StDebuggerExtensionVisitor`s to visit its debugger extensions to display information from these extensions anywhere in the debugger.
For now, `StDebuggerExtensionVisitor` has one concrete subclass `StDebuggerExtensionInspectorNodeBuilderVisitor` that visits debugger extensions to add inspector nodes in the debugger inspector.
This architecture follows the `Visitor` pattern so a debugger extension defines a method `#accept`  that does nothing by default (Figure *@fig:debugger-extension-accept@*).

![Debugger extension's `#accept` method](./graphics/debugger-extension-accept.png width=70&label=fig:debugger-extension-accept)

Following the pattern, you could override this method to call a `visitXXX` method (Figure *@fig:debugger-extension-accept-override@*) that you could define in `StDebuggerExtensionVisitor` (Figure *@fig:debugger-extension-visit-definition@*) and override in subclasses to add information from your extension in the debugger.

```language=Pharo&caption=Overriding #accept method for your debugger extension&label=fig:debugger-extension-accept-override
StMyDebuggerExtension>>#accept: aVisitor

	aVisitor visitMyDebuggerExtension: self
```

```language=Pharo&caption=Adding a #visit method for your debugger extension in the base class of debugger extension visitors&label=fig:debugger-extension-visit-definition
StDebuggerExtensionVisitor>>#visitMyDebuggerExtension: aStMyDebuggerExtension


```



