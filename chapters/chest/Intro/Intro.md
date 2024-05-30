### Chest's Introduction

_Chest_ is a tool providing an _API_ and a Graphical User Interface (_GUI_) to store objects from a program A and to access it from a program B. 
For example, this can be used to keep to check equality between objects from the two programs...

#### Chest's Repositories

Originally, this project was the idea of the former PhD student _Thomas Dupriezt_.
[Here is the link to the original repository](https://github.com/dupriezt/Chest).
Now, the project is maintained by the [_pharo-spec_ organization](https://github.com/pharo-spec/Chest).

#### Install Chest

```smalltalk
Metacello new
    baseline: 'Chest';
    repository: 'github://pharo-spec/Chest';
    load.
```

#### Chest's Basics

Each `Chest` instance has a unique ID (String), which means that two chests cannot have the same ID.
Each `Chest` class has a default instance and it is possible to send messages to these `Chest` classes as if they were their default instance themselves.
Instances of `Chest` hold strong references to stored objects, which means that they prevent these objects from being garbage-collected, whereas instances of `WeakChest` hold weak references to stored objects, which means that they do not prevent these objects from being garbage-collected.

#### Open Chest

_Chest_'s main _GUI_ can be opened **via an entry in the "Debug" world menu** of Pharo (Figure *@fig:chest-debug-menu@*).

![_Chest_ in _Debug_ World Menu](./figures/chest-debug-menu.png width=70&label=fig:chest-debug-menu)

_Chest_ can also be enabled as a _debugger extension_ in the debugging settings of Pharo (Figure *@fig:chest-extension-settings@*).

![_Chest_ Debugger Extension in the Pharo Settings](./figures/chest-extension-in-settings.png width=90&label=fig:chest-extension-settings)



