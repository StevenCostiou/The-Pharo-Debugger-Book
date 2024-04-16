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




