### Opening Debuggers

There are many ways to open debuggers.
In the normal case, exceptions automatically trigger the opening of a debugger.
For example, executing `1/0` will raise a `ZeroDivide` exception and opens a debugger.
Breakpoints and the `self halt` instruction raise special exceptions known as `Break` and `Halt` and open debuggers.
The other way to open debuggers is by debugging a specific piece of code.
This can be done by selecting that code, doing a right click and then selecting _Debug it_ from the contextual menu, or by using the debug shortcut (_ctrl+shift+d_ on windows/linux or _cmd+shift+d_ on mac).
You can open any number of debuggers at the same time.
For example, you can debug two programs side by side, or leave a debugger open while doing other activities --- such as debugging another piece of code.

Once you are in a debugger, you can observe and control your execution. 
In the following, we describe what you will see in a debugger, and what you will be able to do.



